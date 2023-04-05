---
layout: post
title:  "Multidevice previews in Xcode and Android Studio"
date:   2023-04-05 09:00:00
categories: 
    - ios
    - swiftui
    - android
    - compose
permalink: /blog/:title
---

Developing with SwiftUI and Jetpack Compose is very user-friendly. Both frameworks offer a highly readable declarative syntax and a live preview feature that updates in real-time while coding. This live preview allows developers to easily configure various device settings, although it can be a bit cumbersome to specify multiple device factors. At Playtomic, we have created a simple utility for both platforms that allows us to run our previews on a predefined set of devices. Here's what we did:

## Android
This platform was the easy one, as it already supports aggregated annotations out of the box. All you need to do is include somewhere in your project:

```
@Preview(showSystemUi = true, device = Devices.PIXEL_4_XL, name = "Pixel 4 XL")   // big xxxhdpi
@Preview(showSystemUi = true, device = Devices.NEXUS_5, name = "Nexus 5")   // small-medium xxhdpi
annotation class MultiDevicePreview
```
And then just replace the usage of `@Preview` by `@MultiDevicePreview` in your previews:

```
@MultiDevicePreview
@Composable
private fun AMultiDevicePreview() {
    Text("Multidevice preview")
}
```

![Screenshot from preview in Android Studio using multiple device previews](/images/posts/45/androidstudio.png){:class="img-responsive"}


## iOS
Things become a bit more complicated for iOS. Firstly, Xcode only recognizes previews that implement the `PreviewProvider` protocol. As a result, extending the interface with our custom version of `MultiDevicePreviewProvider` doesn't work. Instead, we created a second protocol to tackle this problem:

```
public protocol MultiDevicePreview {
    associatedtype DevicePreviews : View
    @ViewBuilder @MainActor static var devicePreviews: DevicePreviews { get }

    @MainActor static var devices: [PreviewDevice] { get }
    @MainActor static var previewName: String? { get }
}
```

And then, we provided a default implementation of the previews for those using the new protocol:

```
public extension PreviewProvider where Self: MultiDevicePreview {
    static var previews: some View {
        ForEach(devices) { device in
            AnyView(devicePreviews)
                .previewDevice(device)
                .previewDisplayName([previewName, device.rawValue].compactMap { $0 }.joined(separator: " - "))
        }
    }

    static var devices: [PreviewDevice] { PreviewDevice.allCases }
    static var previewName: String? { String(describing: Self.self) }
}
```

We also declared some predefined list of devices to simplify the management:

```
extension PreviewDevice {
    public static let iPhone14 = PreviewDevice("iPhone 14")
    public static let iPhone14Max = PreviewDevice("iPhone 14 Pro Max")
    public static let iPhoneSE = PreviewDevice("iPhone SE (3rd generation)")
    public static let allCases = [iPhone14, iPhone14Max, iPhoneSE]
}

extension PreviewDevice: Identifiable {
    public var id: String {
        rawValue
    }
}
```

With all of the above, you can make use of multidevice previews by conforming your preview to `MultiDevicePreview` and replace the method `previews` by `devicePreviews` like:

```
struct AMultiDevicePreview: PreviewProvider, MultiDevicePreview {
    static var devicePreviews: some View {
        Text("multidevice preview text")
    }
}
```
![Scheenshot from Xcode using multiple device previews](/images/posts/45/xcode.png){:class="img-responsive"}

Not a huge difference, but a nice small addition to the toolkit!