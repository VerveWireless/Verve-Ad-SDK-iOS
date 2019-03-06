# Verve-Ad-SDK-iOS

## Integrating the SDK Framework
Add the provided framework file to your application by adding it as an Embedded Binary to your target. You can do this by pressing the "+" button under the "Embedded Binaries" section under the "General" tab of your application's target or by dragging and dropping the framework file into this section. If dragging and dropping be sure to move the framework file into the same file as your Xcode project. Check "Copy Items If Needed" when prompted.


### App ID
Verve should have sent you a unique identifier for your application. This identifier should be used to fill in the fields labelled `appID`. If you do not have an `appID`, please contact your Verve Publisher Services account manager.

### XML Library
You will also need to update a couple of Build Settings for your app's target:

`Header Search Paths` needs to contain `/usr/include/libxml2`

`Other Linker Flags` needs to contain `-lxml2`

### App Transport Security
The Ad SDK will need to communicate with HTTP endpoints to function. To allow this you will have to add a value to the `info.plist` file associated with your application's target. To do this, click on your application's project in Xcode, then select the app's target. Click the `info` tab and add a new key to the `Custom iOS Target Properties` called `App Transport Security Settings`. This should create a dictionary, to which you should add the key `Allow Arbitrary Loads` and set the value for that key to `YES`.

### Testing
Verve ads can only be shown to IP addresses located inside the US. If you are testing outside of the US, please utilize a proxy or VPN on your device to ensure delivery of test campaigns.

## Interstitial Video
An Interstitial Ad must be loaded and shown on a UIViewController. This VC should also conform to the VRVInterstitialAdDelegate protocol.

### Import Headers and Set AppID
First, import the Interstitial Ad header by calling `#import <verveAdSDK/VRVInterstitialAd.h>` with your other import statements.

Then, use `+ (void)setInterstitialAdDelegate:(UIViewController<VRVInterstitialAdDelegate> *)delegate appID:(NSString *)appID;`
to set the VC that will show the ad.

>Interstitial Ads will present in their own UIViewController, however a root View Controller must be designated. This root VC should be the active VC when attempting to present an ad.

### Loading an Interstitial Ad
To load an Interstitial Ad call `+ (void)loadInterstitialAdForZone:(NSString *)zone;` and pass in the desired zone. Only one ad per zone will be loaded.

Once an ad is ready to be shown, the delegate function `- (void)onInterstitialAdReadyForZone:(NSString *)zone;` will be called.

### Show an Interstitial Ad

After loading successfully, you can safely call `+ (void)showInterstitialAdForZone:(NSString *)zone;`, passing in the relevant zone, to show the ad.

After successfully closing the ad once it's been viewed, `- (void)onInterstitialAdClosedForZone:(NSString *)zone;` will be called.

### Interstitial Delegate Callbacks - VRVInterstitialAdDelegate
```objc
    // Called when ad has been successfully loaded, and is ready to be shown
- (void)onInterstitialAdReadyForZone:(NSString *)zone;

    // Called when no ad is available to be shown, or an ad has failed to load
- (void)onInterstitialAdFailedForZone:(NSString *)zone;

    // Called when ad has been closed and dismissed from view
- (void)onInterstitialAdClosedForZone:(NSString *)zone;
```

### Sample Code
```objc
#import "ViewController.h"
#import <verveAdSDK/VRVInterstitialAd.h>

@interface ViewController () <VRVInterstitialAdDelegate>
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [VRVInterstitialAd setInterstitialAdDelegate:self appID:@"6cFM0mTUUh"];
    [VRVInterstitialAd loadInterstitialAdForZone:@"vast2"];
}

- (void)onInterstitialAdReadyForZone:(nonnull NSString *)zone {
    if ([zone isEqualToString:@"vast2"]) {
      [VRVInterstitialAd showInterstitialAdForZone:zone];
    }
}

@end
```


## Rewarded Video
Rewarded Ads work in exactly the same manner as Interstitial ads, only they use the function calls included in `<verveAdSDK/VRVRewardedAd.h>`. If using both Interstitial and Rewarded ads keep in mind that only one delegate can be active at a time between these two types of full screen ads. Calling `+ (void)setRewardedAdDelegate:(UIViewController<VRVRewardedAdDelegate> *)delegate appID:(NSString *)appID;` will remove the Interstitial ad delegate and vice versa.

### Import Headers and Set AppID
First, import the Rewarded Ad header by calling `#import <verveSDK/VRVRewardedAd.h>` with your other import statements.

Then, use `+ (void)setRewardedAdDelegate:(UIViewController<VRVRewardedAdDelegate> *)delegate appID:(NSString *)appID;`
to set the VC that will show the ad.

### Loading a Rewarded Video Ad
To load an Rewarded Video Ad call `+ (void)loadRewardedAdForZone:(NSString *)zone;` and pass in the desired zone. Only one ad per zone will be loaded.

Once an ad is ready to be shown, the delegate function `- (void)onRewardedAdReadyForZone:(NSString *)zone;` will be called.

### Show a Rewarded Video Ad

After loading successfully, you can safely call `+ (void)showRewardedAdForZone:(NSString *)zone;`, passing in the relevant zone, to show the ad.

After successfully closing the ad once it's been viewed, `- (void)onRewardedAdClosedForZone:(NSString *)zone;` will be called.

### Rewarded Video Delegate Callbacks - VRVRewardedAdDelegate
The only significant way that Rewarded Ads differ from Interstitial ads in their execution is the delegate call `- (void)onRewardedAdRewardedForZone:(NSString *)zone;`. This delegate function will be called when a rewarded ad has reached the point where the user has met the requirements for the reward.

```objc
    // Called when ad has been successfully loaded, and is ready to be shown
- (void)onRewardedAdReadyForZone:(NSString *)zone;

    // Called when no ad is available to be shown, or an ad has failed to load
- (void)onRewardedAdFailedForZone:(NSString *)zone;

    // Called when a user has completed the ad view, and has earned the reward
- (void)onRewardedAdRewardedForZone:(NSString *)zone;

    // Called when ad has been closed and dismissed from view
- (void)onRewardedAdClosedForZone:(NSString *)zone;
```

## Banner Ads
To create a banner ad instantiate a VRVBannerAdView object using
```objc
- (instancetype)initWithDelegate:(id<VRVBannerAdDelegate>)delegate
                           appID:(NSString *)appID
                      bannerSize:(VRVBannerAdSize)adSize
                       andRootVC:(UIViewController *)rootVC;
```

This view will automatically be sized to fit the width of the screen in portrait mode according to the following ratios for different VRVBannerAdSize's:

| Size          | Constant           | Description          |
| ------------- |:------------------:| --------------------:|
| 320x50        | VRVBannerSizeBanner| Standard banner      |
| 728x90        | VRVBannerSizeTabletBanner	     | IAB Leaderboard      |
| 300x250	    | VRVBannerSizeMedRectangle   | IAB Medium Rectangle |

### Loading a Banner Ad
After creation, call `+ (void)loadAdForZone:(NSString *)zone;` on the created VRVBannerView object to load an ad, which will display in the view as soon as it's loaded.

### VRVBannerAdDelegate / RootVC
A delegate must be attached to a banner ad which conforms to the VRVBannerAdDelegate protocol. Additionally, a reference to the UIViewController (rootVC) that will hold the banner ad must be passed to it. The delegate and rootVC can be the same object.

```objc
    // Called when ad has been successfully loaded
- (void)onBannerAd:(VRVBannerAdView *)bannerAd readyForZone:(NSString *)zone;

    // Called when no ad is available to be shown, or an ad has failed to load
- (void)onBannerAd:(VRVBannerAdView *)bannerAd failedForZone:(NSString *)zone;

    // Called when the banner ad will close
- (void)onBannerAd:(VRVBannerAdView *)bannerAd closedForZone:(NSString *)zone;
```


### Scroll View Reference
Some ads will change behavior according to the scrolling of a view. To use this feature you must call the function `- (void)informBannerAdOfScrollViewEvent:(UIScrollView *)scrollView;` on the banner view object when this scroll view scrolls. This is generally done within the function `- (void)scrollViewDidScroll:(UIScrollView *)scrollView`, which is a delegate call for `UIScrollViewDelegate`.

>Please note that it is the application's responsibility to add the VRVBannerView as a subview and position it.

### Sample Code:
```objc
#import "ViewController.h"
#import <verveAdSDK/VRVBannerAdView.h>

@interface ViewController () <VRVBannerAdDelegate, UIScrollViewDelegate>

@property VRVBannerAdView *bannerView;
@property UIScrollView *scrollView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    self.scrollView = [[UIScrollView alloc] initWithFrame:CGRectMake(0, 0, 100, 100)];
    [self.view addSubview:self.scrollView];

    self.bannerView = [[VRVBannerAdView alloc] initWithDelegate:self
                                                          appID:@"6cFM0mTUUh"
                                                     bannerSize:VRVBannerSizeBanner
                                                      andRootVC:self];
    [self.view addSubview:self.bannerView];
    [self.bannerView loadAdForZone:@"banner"];
}

- (void)onBannerAd:(VRVBannerAdView *)bannerAd readyForZone:(NSString *)zone {
    if ([zone isEqualToString:@"banner"] && [bannerAd isEqual:self.bannerView]) {
        NSLayoutConstraint *bottomBanner = [NSLayoutConstraint constraintWithItem:bannerAd
                                                                        attribute:NSLayoutAttributeBottom
                                                                        relatedBy:NSLayoutRelationEqual
                                                                           toItem:self.view.layoutMarginsGuide
                                                                        attribute:NSLayoutAttributeBottom
                                                                       multiplier:1
                                                                         constant:0];
        [self.view addConstraint:bottomBanner];
    }
}

- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
    if (self.bannerView) {
        [self.bannerView informBannerAdOfScrollViewEvent:scrollView];
    }
}

@end
```
