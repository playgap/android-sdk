# Playgap Android SDK

[![Maven Central](https://img.shields.io/maven-central/v/io.playgap/playgap-sdk.svg)](https://central.sonatype.com/artifact/io.playgap/playgap-sdk)

The Playgap SDK enables mobile app developers to monetize offline and online gameplay via advertisements.

### Supported Platforms

- [Unity](https://github.com/playgap/unity-plugin)
- [iOS](https://github.com/playgap/ios-sdk)
- [Android](https://github.com/playgap/android-sdk)

## How to use the SDK

1. Initialize Playgap SDK at the start of every session while offline or online
2. Load and Show Playgap Ads to users
3. Grant rewards for completed ads
4. Grant additional rewards for Claiming Rewards from Offline Ads when the user returns online

## How to Integrate the SDK

<details>
  <summary><b>Requirements and Integration Steps</b></summary>

## Requirements

| Platform | Minimum API Version | Minimum Gradle version|
|----------|---------------------|-----------------------|
| Android  | 21                  | 7.2                   |

## Integration
Playgap SDK is bundled together in an Android Archive (AAR) file that you can use as a dependency for an Android app module.
Unlike JAR files, AAR files can contain Android resources and a manifest file, which lets you bundle in shared resources like layouts and drawables in addition to Kotlin or Java classes and methods.

### Add public Maven distribution to Gradle
Playgap SDK is published on a public Maven distribution [Sonatype Maven Central](https://central.sonatype.com/).

You need to add the `mavenCentral()` maven artefact repository to your `settings.gradle` (if using Gradle version 7+).

```kotlin
dependencyResolutionManagement {
    repositories {
        mavenCentral()
    }
}
```

### Add AAR to Gradle
To add the AAR dependency to your project, import the latest version of the Playgap SDK AAR to your `build.gradle`.  
You're also encouraged to add the Android Advertising ID (AAID) dependency.

If you're using Kotlin DSL `kts`:
```kotlin
android {
    ...
}
dependencies {
    implementation ("io.playgap:playgap-sdk:latestVersion")
    implementation ("com.google.android.gms:play-services-ads-identifier:latestVersion")
}
```

Playgap SDK `latestVersion`: [![Maven Central](https://img.shields.io/maven-central/v/io.playgap/playgap-sdk.svg)](https://central.sonatype.com/artifact/io.playgap/playgap-sdk)  
You can get the `latestVersion` for Android Advertising ID (AAID) dependency on [Google Maven](https://maven.google.com/web/index.html#com.google.android.gms:play-services-ads-identifier).

### Android Manifest

In order for Playgap SDK to work correctly, you need to add the following permissions to your `AndroidManifest.xml`:

```xml
<!-- For getting the Android Advertising ID (AAID) where applicable -->
<uses-permission android:name="com.google.android.gms.permission.AD_ID" />
```

#### Permissions already present in the Playgap SDK's Android Manifest

Other permissions are already added within the Playgap SDK's `AndroidManifest.xml` and you DON'T need to add them explicitly as the manifests will be merged into a common one during build time:
```xml
<!-- For creating HTTP requests -->
<uses-permission android:name="android.permission.INTERNET" />
<!-- For monitoring network state, used for offline/online flow switching -->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<!-- For monitoring network state, used for offline/online flow switching -->
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
```

</details>

## What setups do the SDK work with <a name="how-to-use"></a>

<details>
  <summary><b>Rewarded and Interstitial Placements</b></summary>
    <p></p>

Playgap Ads can be showed for all ad scenarios (Rewarded and Interstitial) within your app. This includes examples such as:
- A user pressing a button offering 20 in-game currency to watch an ad
- An ad automatically shown after failing to complete a level

**The user must receive a Reward for viewing these ads.**

You can show Playgap Ads in both Rewarded Ad surfacing points and Interstitial Ad surfacing points within the same application. This would cover both cases of User-Initiated and Application-Initiated Ads.

The APIs and Configurable features offered by the SDK allow for the game to have full discretion over the manner in which ads are displayed to the user.

</details>
<details>
  <summary><b>Offline Gameplay</b></summary>
    <p></p>

During offline gameplay, the Playgap SDK can be used to Initialize the SDK, Load, and Show ads, and Check Rewards.

The SDK must be initialized at least once while connected to the internet first to be able to be used for offline sessions.

After this, ads can be shown offline to keep the experience consistent for users, and users should receive rewards for completed offline ads. Ads that complete fully offline are considered Unclaimed. If the user reconnects to the internet while the ad is playing, then the SDK will automatically shift to the Online Flow, and grant the reward as Claimed if the Appsheet is able to be presented during this flow.

Once the user returns to online gameplay, the application should present the user with a dialog to claim the Unclaimed rewards as soon as possible.

On this dialog, upon claiming the reward, the user will be presented with Appsheets associated with the ads that were watched offline. Playgap counts the rewards as Claimed after this Appsheet is dismissed.

It is imperative that the Appsheets are shown, as this is what is counted as a monetizeable event.

We recommend that this dialog is displayed via the Claim Rewards API as soon as the connection is re-established for a user with unclaimed rewards. The Claim Rewards sheet can be launched whenever the user is connected to the internet.

**The user should be rewarded for both completing ads while offline, as well as when they claim rewards while online.**

</details>

<details>
  <summary><b>Online Gamplay</b></summary>
    <p></p>

During online gameplay, the Playgap SDK can be used to Initialize the SDK, Load, and Show ads, and Check Rewards.

In comparison to the offline flow, online ads will display the Appsheet to the user during the flow. If the user loses connection before this point, then it will fallback to the Offline flow.

Ads which are able to display the Appsheet while the user is connected will have their rewards immediately counted as Claimed. Claimed rewards do not require the Claim Reward screen to later be shown while online.

</details>

## SDK APIs <a name="sdk-apis"></a>

<details>
  <summary><b>Initialize</b></summary>

### `PlaygapAds.initialize`

The Initialize API prepares the SDK for use during a user session. To use the it effectively:
1. Call Initialize immediately once the application launches with the correct API Key:
- Use `PlaygapTestID123` for testing to always receive test ads
- Obtain your own API Key [steps to go live](#go-live) with your application
2. Await for Initialization Complete to use the other SDK APIs
3. Avoid calling Initialize multiple times (such as in the Unity Update Loop)

```kotlin
import io.playgap.sdk.PlaygapAds

class YourApplication: Application() {

    override fun onCreate() {
        super.onCreate()
        PlaygapAds.initialize(
            context = this,
            apiKey = "YOUR_API_KEY",
            listener = object : InitializationListener {
                override fun onInitializationError(error: InitError) {
                  Log.e("Initialization failed - ${error.type.description(context)}", error)
                }
    
                override fun onInitialized() {
                  Log.i("Initialization completed")
                }

            }
        )
    }
}
```

</details>

<details>
  <summary><b>Load</b></summary>

### `PlaygapAds.loadRewarded`

The LoadRewarded API prepares a fullscreen ad to be shown during the user session. To use the it effectively:
1. Call LoadRewarded before you need to show an Ad
2. Await for a successful LoadComplete to return a single-use Playgap Ad

```kotlin
var loadedAd: PlaygapAd?


PlaygapAds.loadRewarded(
    context = context,
    listener = object : LoadListener {
        override fun onLoadFailed(error: LoadError) {
            Log.e("Load failed - ${error.type.description(context)}", error)
        }
  
        override fun onLoaded(ad: PlaygapAd) {
            loadedAd = ad
            Log.i("Load successful triggered with id: ${ad.objectId}")
        }
    }
)
```

</details>

<details>
  <summary><b>Show</b></summary>

### `PlaygapAds.show`

The Show API prepares a fullscreen ad to be shown during the user session. To use the it effectively:
1. Ensure LoadRewarded returned a Playgap Ad
2. Call Show when you want to show a Fullscreen Ad
3. Reward the user for completing the Ad
- Offline: Only `onShowCompleted` will be triggered
- Online: Both `onUserClaimedReward` and `onShowCompleted` will be triggered
4. Resume normal app operation when either `onShowSkipped` or `onShowCompleted` is triggered


```kotlin
val ad = loadedAd ?: return

PlaygapAds.show(
    context = context,
    ad = ad, 
    listener = object : ShowListener {
        override fun onShowFailed(error: ShowError) {
          Log.e("Show failed - ${error.type.description(context)}", error)
        }
      
        override fun onShowSkipped() {
          Log.i("Show skipped by user")
        }
      
        override fun onShowImpression(impressionId: String) {
          Log.i("Show impression")
        }
      
        override fun onShowPlaybackEvent(event: PlaybackEvent) {
          Log.i("Show playback event - $event")
        }
      
        override fun onShowCompleted(reward: Reward) {
          Log.i("The video ad was closed")
        }
      
        override fun onUserClaimedReward(reward: Reward) {
          Log.i("User claimed reward")
        }
    }
)
```

</details>

<details>
  <summary><b>Check Rewards</b></summary>


### `PlaygapAds.checkRewards` <a name="check-rewards"></a>

The CheckRewards API is a utility function to understand the internal state of Rewards in the SDK. To use the it effectively:
1. Read the "Understand How ID Tracking Works" under [Go-Live Checklist](#go-live-checklist)
2. Test and Trigger the scenarios mentioned in this section through your application code
3. Display a UI button within your application when the user has unclaimed rewards and is connected to the internet

```kotlin
val rewards = PlaygapAds.checkRewards()
Log.i("Unclaimed rewards ${rewards.unclaimed}")
Log.i("Claimed rewards ${rewards.claimed}")

if (!rewards.unclaimed.isEmpty) {
  // Show UI button with amount of Unclaimed rewards
}
```

</details>

<details>
  <summary><b>Claim Rewards</b></summary>

### `PlaygapAds.claimRewards`<a name="claim-rewards"></a>

The ClaimRewards API is used to present a Dialog to the user which allows them to Claim their Unclaimed Rewards when they return online. To use the it effectively:
1. Read and Understand what Unclaimed Rewards are under [PlaygapAds.CheckRewards](#check-rewards)
2. Call ClaimRewards as soon as possible once a user establishes internet connection (see [PlaygapAds.NetworkObserver](#network-observer))

```kotlin
PlaygapAds.claimRewards(
    context = activity,
    listener = object : ClaimRewardsListener {
        override fun onRewardScreenShown() {
            Log.i("Claim reward screen shown")
        }
    
        override fun onRewardScreenFailed(error: ClaimRewardError) {
            Log.e("Claim reward screen failed - ${error.type.description(context)}", error)
        }
    
        override fun onRewardScreenClosed() {
            Log.i("Claim reward screen closed")
        }
    
        override fun onStoreClick() {
            Log.i("Google Play store app sheet clicked")
        }
    
        override fun onUserClaimedReward(reward: Reward) {
            Log.i("User claimed reward")
        }
    }
)
```

</details>

<details>
  <summary><b>Network Observer</b></summary>

### `PlaygapAds.networkObserver`<a name="network-observer"></a>

The Network Observer is a utility which exposes the connection state of the user and provides updates the moment that a connection change occurs. To use it effectively:
1. Attach an observer to it when the application launches
2. Call [PlaygapAds.ClaimRewards](#claim-rewards) as soon possible in the Application Flow when the Network Observer produces a `true` value (meaning the user has reconnected to the internet)

```kotlin
import io.playgap.sdk.PlaygapAds

class YourApplication: Application() {
    
    var isUserConnected: Boolean = true

    override fun onCreate() {
        super.onCreate()
        PlaygapAds.networkObserver(
            context = this,
            observer = object: NetworkObserver {
                override fun onNetworkChange(isConnected: Boolean) {
                    isUserConnected = isConnected
                    Log.i("Is connected to network ${isConnected}")
                }
            }
        )
    }
}
```

</details>

## How to go live with the SDK <a name="go-live"></a>

1. Integrate the SDK using the [Go-Live Checklist](#go-live-checklist) as a guide
2. Obtain an API Key for you application by reaching out us to via hello@playgap.io
3. Exchange the Testing API key for your Application API Key
4. Submit your application to the App Store

## Go-Live Checklist <a name="go-live-checklist"></a>

Below are necessities and knowledge required to get the maximum effective from of the Playgap SDK. Without abiding by these concepts, your application will not be as performant.

<details>
  <summary><b>Initialize the SDK as soon as the App Launches</b></summary>
    <p></p>

When your application launches, initialize the Playgap SDK immediately.

It is a common user behavior to disconnect from the internet during the first few seconds of gameplay. It's vital to use this window of opportunity to ensure that the Playgap SDK can use this opportunity to prepare and update ads to present to the user while offline.

</details>

<details>
  <summary><b>Claim Rewards as soon as Connection is Available</b></summary>
    <p></p>

The Network API gives access to the moment a user reconnects to the internet. This should be done through a combination of:
- Automatically displaying the Claim rewards screen once the user reconnections
- And; Displaying a UI button within the application showing the amount of unclaimed rewards

Displaying users with the opportunity to install the ad while online is required to generate revenue. Therefore, it is important that the Claim Rewards Dialog is displayed to users when they have claimed rewards while offline, and while the connection to a network is established.

The amount of unclaimed rewards can be accessed via the Check Rewards API after the SDK has been initialized successfully.

</details>

<details>
  <summary><b>Rewarding the User</b></summary>
    <p></p>

Ensure that the user receives rewards both for completing offline views as well as reclaiming rewards online.

Decent rewards must be given to the user, otherwise they will not be incentivized to complete ads or claim rewards when available.

It's also quite important to give users an indication of what they've been rewarded with. This is most commonly done through simple animations after the Claim Rewards screen closes.

</details>

<details>
  <summary><b>Working with Online Mediation</b></summary>
    <p></p>

If your application is integrated with any Online Mediation SDK, such as MAX, LevelPlay, or AdMob, then you should implement the following:
1. When the user is online, call Playgap to Show Ads only when Mediation fails to Load a suitable ad
2. When the user is offline, only call Playgap to Show Ads, even if an ad was loaded from Online Mediation before the user goes offline.
- Online Mediation SDKs assume that the user is connected to the internet when watching an ad, which will cause breaking flows within your application

</details>

<details>
  <summary><b>A/B Testing</b></summary>
    <p></p>

Where possible, it's important to A/B test your application's integration with Playgap where possible for two reasons:
- If there are any errors in the integration, then the integration can be resolved prior full rollout
- When the integration is valid, it will be possible to see the overall growth of revenue of all user segments

The expectation is that there will be overall revenue growth of your Online user segment, as well as direct revenue growth from the Offline user segment.

</details>

<details>
  <summary><b>Understand How ID Tracking Works</b></summary>
    <p></p>

The Playgap SDK outputs certain identifiers on:

- The Loaded Playgap Ad => Object ID
- The Impression once the ad is shown => Impression ID
- The Reward collected offline when the ad completes => Reward ID
    - If the user was offline, the Playgap SDK considers this an "Unclaimed" Reward
    - If the user was online, the Playgap SDK considers this an "Claimed" Reward
- The Claimed Reward once the user returns online => Reward ID
    - At this stage, the Playgap SDK considers this a "Claimed" Reward

All of these IDs will be identical at the different stages of the ad lifecycle, and are exposed via the SDK APIs. These can be used for any purpose, such as **event tracking, fraud prevention, or unique rewarding solutions.**

The PlaygapAds.CheckRewards API can used to check both claimed and unclaimed rewards:

- Claimed rewards are all of the Reward IDs the user has received since the most recent update of the Playgap SDK
- Unclaimed rewards are all of the rewards from advertisements which have not yet been claimed from the Claim Rewards Dialog

</details>

## Need additional help?

If you have any questions, then our engineering and business teams would be happy to support you over email and messengers via hello@playgap.io or directly through your account manager.
