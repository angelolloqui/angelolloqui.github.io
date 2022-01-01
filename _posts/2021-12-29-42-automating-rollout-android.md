---
layout: post
title:  "Automating rollout releases in Android"
date:   2021-12-29 12:00:00
categories: 
    - android
    - googlePlay
    - automation
    - bitrise    
permalink: /blog/:title
---

As part of our new release process (weekly releases) we are also **changing the way we are publishing the apps for users**. Since it is now automatic, it is crucial for us to have a **phased released** in which only a small subset of our users get the latest build, increasing daily and acting as a "failsafe" in case of an important bug makes it into production. 

For iOS, we can set the [Phased Release](https://help.apple.com/app-store-connect/#/dev3d65fcee1) + Publication Date option and the AppStore will handle it in your behalf, starting the release on a certain date and increasing the automatic updates to 1%, 2%, 5%... each day.

However, Google's approach is different. They do not offer a release date nor an automated phased release. Instead, they offer you with an [API](https://developers.google.com/android-publisher/api-ref/rest/v3/edits.tracks) (and web dashboard) where you can set the percentage of users yourself at any time. This is in many senses much better than the Apple one, especially since this actually controls the releases and not just the automatic updates like in Apple, but it has a downside: **it is all manual**.

With our current setup of weekly releases and a phased released across 6 days (2%, 5%, 10%, 20%, 50%, 100%), this basically means having to **update every single day the rollout amount manually**. I am not sure about you, but having to enter every single day to click some button is the last thing I want to do.

![Press the button](https://media.giphy.com/media/ZWbeEcbeo0cKI/giphy.gif)

## Automating the release rollout

So, we wondered, if we have an API to control the rollout percentage, isn't that enough to make it automatic? what if we have a CI job that run every day and basically checks if there is an ongoing rollout release, and in that case increases the percentage? Let's see how we did it:

```
# rollout_update.py

import copy
import sys
import httplib2
from apiclient.discovery import build
from oauth2client.service_account import ServiceAccountCredentials
from oauth2client.client import AccessTokenRefreshError

TRACK = ('production')

# To run: rollout_update package_name json_credentials_path
def main():
  PACKAGE_NAME = sys.argv[1]
  credentials = ServiceAccountCredentials.from_json_keyfile_name(
    sys.argv[2],
    scopes='https://www.googleapis.com/auth/androidpublisher')

  http = httplib2.Http()
  http = credentials.authorize(http)

  service = build('androidpublisher', 'v3', http=http)

  try:
    edit_request = service.edits().insert(body={}, packageName=PACKAGE_NAME)
    result = edit_request.execute()
    edit_id = result['id']

    track_result = service.edits().tracks().get(editId=edit_id, packageName=PACKAGE_NAME, track=TRACK).execute()
    old_result = copy.deepcopy(track_result)

    print("Current status: ", track_result)
    for release in track_result['releases']:
        if 'userFraction' in release:
            rolloutPercentage = release['userFraction']
            if rolloutPercentage == 0:
                print('Release not rolled out yet')
                continue
            elif rolloutPercentage < 0.02:
                release['userFraction'] = 0.02                         
            elif rolloutPercentage < 0.05:
                release['userFraction'] = 0.05
            elif rolloutPercentage < 0.1:
                release['userFraction'] = 0.1
            elif rolloutPercentage < 0.2:
                release['userFraction'] = 0.2
            elif rolloutPercentage < 0.5:
                release['userFraction'] = 0.5
            elif rolloutPercentage < 1.0:
                del release['userFraction']
                release['status'] = 'completed'
            else:
                print('Release already fully rolled out')
                continue        
    if old_result != track_result:
        completed_releases = list(filter(lambda release: release['status'] == "completed", track_result['releases']))
        if len(completed_releases) == 2:
            track_result['releases'].remove(completed_releases[1])

        print("Updating status: ", track_result)
        service.edits().tracks().update(
                    editId=edit_id,
                    track=TRACK,
                    packageName=PACKAGE_NAME,
                    body=track_result).execute()
        commit_request = service.edits().commit(editId=edit_id, packageName=PACKAGE_NAME).execute()
        print('Edit ', commit_request['id'], ' has been committed')    


  except AccessTokenRefreshError:
      raise SystemExit('The credentials have been revoked or expired, please re-run the application to re-authorize')

if __name__ == '__main__':
  main()
```

In order to run this step, you need:
1. Get a Google Developer Service JSON key credentials file. If you have been using some automation tools like Fastlane for uploading the APK to the GooglePlay, you have this already. Otherwise, follow the instructions from [Fastlane Supply setup](https://docs.fastlane.tools/actions/supply/)
2. Install `pipenv` or some dependency manager for python since the script uses `google-api-python-client` and `oauth2client`. You could get them by running: 
```
pipenv install google-api-python-client                            
pipenv install oauth2client
```
3. Run the script:
`pipenv run python rollout_update.py <your_package> <json_credential_path>`

The script will do the following:
1. Create a new [Google Edit](https://developers.google.com/android-publisher/edits)
2. Fetch the production track release info
3. For each release, if it has a rollout in progress, then it increases the rollout percentage to the next "step", where steps are: 2%, 5%, 10%, 20%, 50%, 100%.
4. If changes performed, then commit the Edit

## Connecting the CI
So now that we have the script to increase the rollout, all we need is to schedule it. In our case, we are using [Bitrise](https://www.bitrise.io/), so we decided to schedule a workflow that runs the script every night. We even created a [Bitrise step](https://github.com/angelolloqui/bitrise-step-google-play-rollout-update) in case you want to use it that handles the dependencies and running the script.

```
  update_rollout:
    steps:
    - git::https://github.com/angelolloqui/bitrise-step-google-play-rollout-update.git@master:
        inputs:
        - package_name: com.playtomic
        - service_account_json_key_path: "$BITRISEIO_BITRISEIO_GOOGLEPLAY_SERVICE_ACCOUNT_JSON_URL"
```
Note: Your credentials file should be stored somewhere secured, like the Generic File Sorage of Bitrise

## Halting a failing release?
If a release goes wrong, all you need to do is to go to the Google Dashboard and halt the release as you would normally do with a manually controlled phased release. The script will simply detect the release is halted and will just ignore it.

## What about Hotfixes?
Well, the script makes no assumption on the type of build or release, so it will basically just work for any release, including hotfixes. However, normally in hotfix builds, since they tend to be critical (otherwise we would not make a hotfix but wait for next week build) we normally deploy it to 100% of the user base, so we do not really need this step to run.



