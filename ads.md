# Ads

this document provides an introduction to ads and sample ads flows. For pure function definitions provided by Pi SDK, refer to [Ads section of SDK reference doc](SDK_reference.md#ads).

## Introduction

Pi App Platform gives developers the opportunity to integrate ads into their Pi App and get rewarded. Displaying ads is open to all applications in the Pi ecosystem, but only applications approved by Pi Core Team can be monetized.
There are two type of ads currently supported: **interstitial** and **rewarded**. This documentation will be updated as new features are released.

Technically, interstitial ads can be displayed at any time, but check out our best practices documentation to learn more.

### Interstitial Ads

Interstitial ads are full-screen ads and are meant to be displayed between some kind of content, usually at natural transition points or breaks, such as in between activities or game levels.

### Rewarded Ads

Rewarded ads are full-screen ads and are meant to be displayed to users in exchange for a reward within developer's app. For example if you're developing a game app and player runs out of lives but wants to continue playing, you might give him an option of obtaining extra live for watching a rewarded ad.

Since rewarded ads are meant to reward a user, they can be displayed only if user of an app is authenticated.

## Prerequisites

In order to be rewarded from the Pi Developer Ad Network you must apply for it on the Pi Developer Portal. Once you’ve applied your application will be reviewed by the Pi Core Team.

**Note**: some of the Ads module methods may return slightly different responses based on whether your application has been approved to Pi Developer Ad Network. These differences are always mentioned in the method definitions described in the [SDK reference](SDK_reference.md#ads).

### Developer Ad Network Application

Coming soon...

### Support for Ad Network on user's Pi Browser

Your users may be using old versions of Pi Browser, on which the Developer Ad Network is not available. You can ensure that Ad Network is available by using [`Pi.nativeFeaturesList` SDK method](SDK_reference.md#native-features-list) which returns an array of native features available on given Pi Browser build. In order for the ads to run properly, the native features list array should include `ad_network` string.

```typescript
// you usually would check the ads support ahead of time and store the information
(async () => {
  await Pi.init({ version: "2.0" });
  const nativeFeaturesList = await Pi.nativeFeaturesList();
  const adNetworkSupported = nativeFeaturesList.includes("ad_network");
  // store adNetworkSupported for later use
})();
```

If `ad_network` is missing, and you want to display ads to users, you can encourage them to update the Pi Browser.

When promises returned from `Pi.Ads.requestAd` and `Pi.Ads.showAd` methods resolve with `"ADS_NOT_SUPPORTED"`, its synonymous to `"ad_network"` missing from the native features list array.

For the in-depth definitions of Pi SDK methods used above, go to [SDK reference](SDK_reference.md#ads).

## Implementation

### Basic Usage

The Pi Browser internally manages the ads availability strategy, automatically loading initial ads and reloading them whenever displayed. This aims to ensure that ads would be consistently ready for display. Therefore, in most cases you should be able to display ads to your users by simply calling the `Pi.Ads.showAd()` methods, passing ad type as an argument.

#### Interstitial Ads

Interstitial ads are simple to integrate. They are usually displayed around some transition points. For example, a level-based game app could display interstitial ads every 3 levels.

```typescript
// we pretend that the game app has just reached end of 3rd level according to above example
completeLevel();

if (currentLevel % 3 === 0) {
  await Pi.Ads.showAd("interstitial");
}

startNewLevel();
```

#### Rewarded Ads

Rewarded ads are meant to allow you to reward your users for watching an ad. Sticking to a game app example, if your game gives a user certain amount of lives and he runs out of them, you could offer him to watch an ad in exchange for one extra live.

Rewarded ads are still simple to display, but require some more effort to ensure security and verify if the user should be rewarded. The `Pi.Ads.showAd("rewarded")` method returns a Promise with one additional field `adId`, with which you can verify validity of the Pi SDK response against Pi Platform API.

> **The user might be cheating on your app!**
>
> Displaying a rewarded ad should lead to rewarding a user with some kind of a reward on your app.
> Since users might be running a hacked version of the SDK and intercept your `displayAd('rewarded')` method, you must verify the rewarded status of the ad using Pi Platform API, before rewarding users.
>
> For documentation of Pi Platform API, refer to [Platform API documentation](platform_API.md#verify-a-rewarded-ad-status).
>
> **You should give rewards to your users only if `mediator_ack_status` for given ad is `"granted"`.**

```typescript
const showAdResponse = await Pi.Ads.showAd("rewarded");
if (showAdResponse.result === "AD_REWARDED") {
  // delegate rewarding user to your backend
  // e.g.:
  const result = await rewardUserForWatchingRewardedAd(showAdResponse.adId);
  if (result.rewarded === true) {
    grantUserAdditionalLive();
  }
}
```

### Advanced Usage

While the examples from previous section are enough to have your application start displaying ads to your users, you might find yourself needing to perform additional actions like manually requesting ads or checking ads availability.

In some cases, loading ads may fail or be interrupted. If you detect that an ad is not available, you can try manually calling `Pi.Ads.requestAd()` passing ad type as an argument.

In other cases, you might need to know whether an ad is ready before calling `Pi.Ads.showAd()` method. You can check it using `Pi.Ads.isAdReady()` method passing ad type as an argument.

For the full definitions of Pi SDK methods with all possible response types, check the [SDK reference](SDK_reference.md#ads).

### Interstitial Ads Advanced Usage

```typescript
// we pretend that the game app has just reached end of 3rd level according to previous example

completeLevel();

if (currentLevel % 3 === 0) {
  const isAdReadyResponse = await Pi.Ads.isAdReady("interstitial");

  if (isAdReadyResponse.ready === true) {
    return Pi.Ads.showAd("interstitial");
  }

  const requestAdResponse = await Pi.Ads.requestAd("interstitial");

  if (requestAdResponse.result !== "AD_LOADED") {
    // indicate to user that ad could not be loaded
    return;
  }

  const showAdResponse = await Pi.Ads.showAd("interstitial");

  if (showAdResponse.result !== "AD_CLOSED") {
    // indicate to user that ad could not be displayed
    return;
  }
}

startNextLevel();
```

As you can see, combining the 3 methods you can react to non-happy paths at any step of the ads flow.

### Rewarded Ads Advanced Usage

Assuming that you've been approved to display ads on your app, here's a high level flow of displaying an ad to your user.

1. User visits your app wants to watch an ad in order to be rewarded.
2. Your app shows an ad on your app using the Pi SDK Ads module.

- For documentation of Pi SDK methods, refer to [SDK reference](SDK_reference.md#ads).

3. With `adId` obtained from the SDK method, you need to verify the status of the ad from the Pi Server. Details are available in the [next section](#rewarded-ads-status-verification-with-pi-platform-api).

- For documentation of Pi Platform API, refer to [Platform API documentation](platform_API.md#verify-a-rewarded-ad-status).

4. Once verified, you can reward the user.

### Rewarded ads status verification with Pi Platform API

> **The user might be cheating on your app!**
>
> Displaying a rewarded ad should lead to rewarding a user with some kind of a reward on your app.
> Since users might be running a hacked version of the SDK and intercept your `displayAd('rewarded')` method, you must verify the rewarded status of the ad using Pi Platform API, before rewarding users.
>
> **You should give rewards to your users only if `mediator_ack_status` for given ad is `"granted"`.**

```typescript
const showRewardedAd = async () => {
  try {
    const isAdReadyResponse = await Pi.Ads.isAdReady("rewarded");

    if (isAdReadyResponse.ready === false) {
      const requestAdResponse = await Pi.Ads.requestAd("rewarded");

      if (requestAdResponse.result === "ADS_NOT_SUPPORTED") {
        // display modal to update Pi Browser
        return showAdsNotSupportedModal();
      }

      if (requestAdResponse.result !== "AD_LOADED") {
        // display modal ads are temporarily unavailable and user should try again later
        return showAdUnavailableModal();
      }
    }

    const showAdResponse = await Pi.Ads.showAd("rewarded");

    if (showAdResponse.result === "AD_REWARDED") {
      // reward user logic:
      // usually delegate rewarding user to your backend which would
      // firstly verify `adId` against Pi Platform API, then decide whether to
      // reward the user and rewarded user if the rewarded ad status is confirmed
      // e.g.:
      const result = await rewardUserForWatchingRewardedAd(adId);
      if (result.rewarded === true) {
        showRewardedModal(result.reward);
      } else {
        showRewardFailModal(result.error);
      }
    } else {
      // fallback logic
      showAdErrorModal();
    }
  } catch (err) {
    // good practice to handle any potential errors
  }
};
```

## Best Practices for Displaying Interstitial Ads

1. **Natural Transition Points**: Display interstitial ads at natural transition points in your app, such as between levels in a game or between different sections of content. This minimizes disruption to the user experience.

2. **Frequency**: Avoid displaying interstitial ads too frequently. This can lead to user frustration and may result in users abandoning your app. A good rule of thumb is to show interstitial ads no more than once every few minutes of active use.

3. **Relevance**: Ensure that the ads displayed are relevant to your users. Irrelevant ads can lead to a poor user experience and may reduce the effectiveness of your ad placements.

4. **Load Time**: Preload interstitial ads to ensure they are ready to display when needed. This helps to avoid delays and keeps the user experience smooth.

5. **Clear Close Option**: Provide a clear and easily accessible option for users to close the interstitial ad. This helps to maintain a positive user experience and reduces frustration.

## Best Practices for Displaying Rewarded Ads

1. **Value Exchange**: Clearly communicate the value exchange to users. Let them know what reward they will receive in exchange for watching the ad. This helps to increase user engagement and satisfaction.

2. **Placement**: Place rewarded ads at points in your app where users are likely to be motivated to earn rewards, such as when they run out of lives in a game or need extra resources to continue.

3. **Frequency**: Avoid overwhelming users with too many rewarded ads. Offer them at strategic points where the reward will be most valuable to the user.

4. **User Control**: Allow users to choose whether or not to watch a rewarded ad. Forcing users to watch ads can lead to a negative experience and may result in users abandoning your app.

5. **Reward Delivery**: Ensure that rewards are delivered promptly and reliably after the user watches the ad. This helps to build trust and encourages users to engage with rewarded ads in the future.

## How to Apply for the Developer Ad Network

1. **Access the Developer Portal**: Open the Pi Browser and navigate to the Pi Developer Portal at `pi://develop.pi`.

2. **Register Your App**: If you haven't already, register your app on the Developer Portal by following the instructions provided in the [Developer Portal documentation](developer_portal.md).

3. **Submit an Application**: Once your app is registered, navigate to the Ads section of the Developer Portal and submit an application to join the Developer Ad Network. Provide all required information and details about your app.

4. **Review Process**: Your application will be reviewed by the Pi Core Team. This process may take some time, so please be patient.

5. **Approval and Integration**: If your application is approved, you will receive instructions on how to integrate the Developer Ad Network into your app. Follow these instructions carefully to start displaying monetized ads in your app.
