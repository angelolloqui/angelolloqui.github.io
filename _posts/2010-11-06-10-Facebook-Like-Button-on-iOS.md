---
layout: post
title:  "Facebook Like Button on iOS"
date:   2010-11-06 11:30:34
categories: 
    - facebook
    - ios
    - like button
permalink: /blog/:title
notice: "Some people are still asking me for this. Please, note that this post is too old and that it was a hack at that time, so it is very likely to fail nowadays. Nevertheless, the idea behind this hack should still work: Embeb a webview with the official FB like button and capture the login redirect to display a proper dialog to the user."
---
  
Some days ago a client asked us for including a Like Facebook button in one of his iPad applications. We have previously used Facebook iOS SDK ([https://github.com/facebook/facebook-ios-sdk](https://github.com/facebook/facebook-ios-sdk)) for including things like the user's profile photo, friends, and so on so we were pretty sure that this button would be easy to implement.  

Upppssss, what an error! Facebook iOS API doesn't include a FB Like button, and the Rest API either. The only way that Facebook seems to give to developers is a HTML button or iframe, both of them thinked for being in a web enviroment. Of course we have the chance to include a webview in the iPad app to include this button, but we should take care of the login process and some other issues, so I did some research and I found this:  
  
[http://petersteinberger.com/2010/06/add-facebook-like-button-with-facebook-connect-iphone-sdk/](http://petersteinberger.com/2010/06/add-facebook-like-button-with-facebook-connect-iphone-sdk/)  
  
This web has the solution that I was looking for, but, after including it's code (with a minor change due to a miss method), it didn't work as expected. The FB login dialog opens and then immediatly closes.  
  
### The solution  
Here are the changes and improvements I have done for resolving it, with the complete code:  
  
First, we have to customize a FBDialog for taking care of the login process:

    
    //FBCustomLoginDialog.h
    @interface FBCustomLoginDialog : FBDialog {
    }
    @end
    

    
    //FBCustomLoginDialog.m
    @implementation FBCustomLoginDialog
    - (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
    	NSURL* url = request.URL;
    	if ([[url absoluteString] rangeOfString:@"login"].location == NSNotFound) {
    		[self dialogDidSucceed:url];
    		return NO;
    	} 
    	else if (url != nil) {
    		[_spinner startAnimating];
    		[_spinner setHidden:NO];
    		return YES;
    	}
    	return NO;
    }
    @end
    

  

As you may see, I changed the method "containsString" by "rangeOfString" because the first doesn't exists on iOS SDK. Another minor change I made is that I included a few sentences for start the spinner animation after the user submits the login form, and I had to put and extra "if" because sometimes the URL is null and it can't be loaded.  
  
OK, so now we have a customized login view, but we need to make the button view for the "I like" UI. In order to build this view, my first option was to inherit my view directly from UIWebView, but after a while I decided to use a UIView over it because it gives me the chance to disable the webview scroll (the webpage generated for the FB Like button usually is heigher that the view itself, resulting in an awful scroll).  
I also made some additions for customizing a little the colors showed in the webview, but I found a problem because the CSS of the page are loaded asynchronously with AJAX, so I can't know the exact moment in which I have to inject my javascript that changes the colors. This may be improved overriden the AJAX onreadystate to do it at the exact moment, but I didn't have so much time to investigate on this to only change a color, so I finally took the easy way, injecting the javascript a while after the page loads (3 secs in the code below). If you find a better solution for this I would appreciate if you post your code :)  
  
Anyway, this is my final code:  
 

    
    //FBLikeButton.h
    #define FB_LIKE_BUTTON_LOGIN_NOTIFICATION @"FBLikeLoginNotification"
    
    typedef enum {
    	FBLikeButtonStyleStandard,
    	FBLikeButtonStyleButtonCount,
    	FBLikeButtonStyleBoxCount
    } FBLikeButtonStyle;
    
    typedef enum {
    	FBLikeButtonColorLight,
    	FBLikeButtonColorDark
    } FBLikeButtonColor;
    
    @interface FBLikeButton : UIView {
    	UIWebView *webView_;
    	UIColor *textColor_;
    	UIColor *linkColor_;
    	UIColor *buttonColor_;
    }
    @property(retain) UIColor *textColor;
    @property(retain) UIColor *linkColor;
    @property(retain) UIColor *buttonColor;
    
    - (id)initWithFrame:(CGRect)frame andUrl:(NSString *)likePage andStyle:(FBLikeButtonStyle)style andColor:(FBLikeButtonColor)color;
    - (id)initWithFrame:(CGRect)frame andUrl:(NSString *)likePage;
    
    @end
    

  

    
    //FBLikeButton.m
    //LoginDialog es estatica para abrir unicamente un login en toda la app
    static FBDialog *loginDialog_;
    
    @implementation FBLikeButton
    
    @synthesize textColor = textColor_, buttonColor = buttonColor_, linkColor = linkColor_;
    
    - (id)initWithFrame:(CGRect)frame andUrl:(NSString *)likePage andStyle:(FBLikeButtonStyle)style andColor:(FBLikeButtonColor)color {
    	if ((self = [super initWithFrame:frame])) {
    		NSString *styleQuery=(style==FBLikeButtonStyleButtonCount? @"button_count" : (style==FBLikeButtonStyleBoxCount? @"box_count" : @"standard"));
    		NSString *colorQuery=(color==FBLikeButtonColorDark? @"dark" : @"light");
    		NSString *url =[NSString stringWithFormat:@"http://www.facebook.com/plugins/like.php?layout=%@&show_faces=true&width=%d&height=%d&action=like&colorscheme=%@&href=%@",
    styleQuery, (int) frame.size.width, (int) frame.size.height, colorQuery, likePage];
    
    		//Creamos una webview muy alta para evitar el scroll interno por la foto del usuario y otras cosas
    		webView_ = [[UIWebView alloc] initWithFrame:CGRectMake(0, 0, frame.size.width, 300)];
    		[self addSubview:webView_];
    		[webView_ loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:url]]];
    		webView_.opaque = NO;
    		webView_.backgroundColor = [UIColor clearColor];
    		webView_.delegate = self;
    		webView_.autoresizingMask = UIViewAutoresizingFlexibleWidth;
    		[[webView_ scrollView] setBounces:NO];
    		self.backgroundColor=[UIColor clearColor];
    		self.clipsToBounds=YES;
    
    		[[NSNotificationCenter defaultCenter] addObserver:webView_ selector:@selector(reload) name:FB_LIKE_BUTTON_LOGIN_NOTIFICATION object:nil];
    	}
    	return self;
    }
    
    - (id)initWithFrame:(CGRect)frame andUrl:(NSString *)likePage{
    	return [self initWithFrame:frame andUrl:likePage andStyle:FBLikeButtonStyleStandard andColor:FBLikeButtonColorLight];
    }
    
    - (void)dealloc {
    
    	[[NSNotificationCenter defaultCenter] removeObserver:webView_ name:FB_LIKE_BUTTON_LOGIN_NOTIFICATION object:nil];
    
    	[webView_ stopLoading];
    	webView_.delegate=nil;
    	[webView_ removeFromSuperview];
    	[webView_ release]; webView_=nil;
    
    	self.linkColor=nil;
    	self.textColor=nil;
    	self.buttonColor=nil;
    
    	[super dealloc];
    }
    
    
    - (void) configureTextColors{
    	NSString *textColor=[textColor_ hexStringFromColor];
    	NSString *buttonColor=[buttonColor_ hexStringFromColor];
    	NSString *linkColor=[linkColor_ hexStringFromColor];
    
    	NSString *javascriptLinks = [NSString stringWithFormat:@"{"
    		"var textlinks=document.getElementsByTagName('a');"
    		"for(l in textlinks) { textlinks[l].style.color='#%@';}"
    		"}", linkColor];
    
    	NSString *javascriptSpans = [NSString stringWithFormat:@"{"
    		"var spans=document.getElementsByTagName('span');"
    		"for(s in spans) { if (spans[s].className!='liketext') { spans[s].style.color='#%@'; } else {spans[s].style.color='#%@';}}"
    		"}", textColor, (buttonColor==nil? textColor : buttonColor)];
    
    	//Lanzamos el javascript inmediatamente
    	if (linkColor)
    		[webView_ stringByEvaluatingJavaScriptFromString:javascriptLinks];
    	if (textColor)
    		[webView_ stringByEvaluatingJavaScriptFromString:javascriptSpans];
    
    	//Programamos la ejecucion para cuando termine
    	if (linkColor)
    		[webView_ stringByEvaluatingJavaScriptFromString:[NSString stringWithFormat:@"setTimeout(function () %@, 3000)", javascriptLinks]];
    	if (textColor)
    		[webView_ stringByEvaluatingJavaScriptFromString:[NSString stringWithFormat:@"setTimeout(function () %@, 3000)", javascriptSpans]];
    }
    
    ///////////////////////////////////////////////////////////////////////////////////////////////////
    #pragma mark -
    #pragma mark UIWebViewDelegate
    
    - (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
    
    	if (loginDialog_!=nil)
    		return NO;
    
    	// if user has to log in, open a new (modal) window
    	if ([[[request URL] absoluteString] rangeOfString:@"login.php"].location!=NSNotFound){
    		loginDialog_= [[[FBCustomLoginDialog alloc] init] autorelease];
    		[loginDialog_ loadURL:[[request URL] absoluteString] get:nil];
    		loginDialog_.delegate = self;
    		[loginDialog_ show];
    		[loginDialog_.delegate retain]; //Retenemos el boton que ha abierto el login para que pueda recibir la confirmacion correctamente
    		return NO;
    	}
    	if (([[[request URL] absoluteString] rangeOfString:@"/connect/"].location!=NSNotFound) || ([[[request URL] absoluteString] rangeOfString:@"like.php"].location!=NSNotFound)){
    		return YES;
    	}
    
    	NSLog(@"URL de Facebook no contemplada: %@", [[request URL] absoluteString]);
    
    	return NO;
    }
    
    - (void)webViewDidFinishLoad:(UIWebView *)webView{
    	[self configureTextColors];
    }
    
    ///////////////////////////////////////////////////////////////////////////////////////////////////
    #pragma mark -
    #pragma mark Facebook Connect
    
    - (void)dialogDidSucceed:(FBDialog*)dialog {
    	[loginDialog_.delegate release];
    	loginDialog_.delegate=nil;
    	loginDialog_=nil;
    
    	//Lanzamos la notificacion para que se actualicen los botones
    	[[NSNotificationCenter defaultCenter] postNotificationName:FB_LIKE_BUTTON_LOGIN_NOTIFICATION object:nil];
    }
    
    /**
    * Called when the dialog succeeds and is about to be dismissed.
    */
    - (void)dialogDidComplete:(FBDialog *)dialog{
    	[self dialogDidSucceed:dialog];
    }
    
    /**
    * Called when the dialog succeeds with a returning url.
    */
    - (void)dialogCompleteWithUrl:(NSURL *)url{
    	[self dialogDidSucceed:loginDialog_];
    }
    
    /**
    * Called when the dialog get canceled by the user.
    */
    - (void)dialogDidNotCompleteWithUrl:(NSURL *)url{
    	[self dialogDidSucceed:loginDialog_];
    }
    
    /**
    * Called when the dialog is cancelled and is about to be dismissed.
    */
    - (void)dialogDidNotComplete:(FBDialog *)dialog{
    	[self dialogDidSucceed:loginDialog_];
    }
    
    /**
    * Called when dialog failed to load due to an error.
    */
    - (void)dialog:(FBDialog*)dialog didFailWithError:(NSError *)error{
    	[self dialogDidSucceed:loginDialog_];
    }
    
    @end
    

  
  

You may notice that I added a lot of code and methods to the original example. Almost all of them are for customizing the UI and make a proper use of our customized login dialog (the FBDialogDelegate methods).  
The only important changes are the use of the NSNotificationCenter for sending notifications when the dialog success (and so every existing Like button could refresh with the new credentials) and the static reference to the FBCustomDialog to avoid multiple login popups at the same time.  
  
The UIColor extensions used on the code above can be found here:  
[http://arstechnica.com/apple/guides/2009/02/iphone-development-accessing-uicolor-components.ars](http://arstechnica.com/apple/guides/2009/02/iphone-development-accessing-uicolor-components.ars)  
  
Finally, you would see a method that doesn't exists on a standard UIWebView:

    
    [[webView_ scrollView] setBounces:NO];
    

  

this method is a little hack I use sometimes to disable the scroll or bounces of the webview, but it may fail on future iOS versions. Anyway, if you still want to use it, this is what it does (I have it in a UIWebView category)

    
    - (UIScrollView *) scrollView {
    	NSArray *subviews = [self subviews];
    	for (UIView *view in subviews){
    		if ([view isKindOfClass:[UIScrollView class]])
    			return (UIScrollView *) view;
    	}
    	return nil;
    }
    

  
  
And thats all! with these 2 classes you are able to include your FB Like buttons in your views in an easy way, just with something like the following:

    
    FBLikeButton *likeButton = [[FBLikeButton alloc] initWithFrame:frame andUrl:@"www.mylikeurl.com"];
    [view addSubview:likeButton];
    [likeButton release];
    

or even with customized colors:

    
    FBLikeButton *likeButton = [[FBLikeButton alloc] initWithFrame:frame andUrl:@"www.mylikeurl.com"];
    [likeButton setTextColor:COLOR_DARK_GRAY];
    [likeButton setLinkColor:COLOR_CLEAR_GRAY];
    [view addSubview:likeButton];
    [likeButton release];
    

  

I hope it helps you all!  
  
  
### Known issues

*   Login Dialog has a double title bar, the FBDialog title and the FB webpage title. I don't think there is an easy solution for this, but I consider it as a minor issue.
*   FBCustomLoginDialog and FBLoginDialog uses different login methods, so the user have to make login twice if you use the FB iOS SDK for other staff. I don't think there is a workaround for this issue.
*   UI colors may change a little time after the button loads, as commented before, and it may stop working on the future if FB changes it's webpage structure.
*   There is no easy way for customizing layout or even alignment of the elements.
*   UIWebview have a little render delay as it loads remote content.