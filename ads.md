# Ads

this document provides an introduction to ads and sample ads flows. For pure function definitions provided by Pi SDK, refer to [Ads section of SDK reference doc](SDK_reference.md#ads).

## Table of Contents

- [Introduction](#introduction)
  - [Interstitial Ads](#interstitial-ads)
  - [Rewarded Ads](#rewarded-ads)
  - [Banner Ads](#banner-ads)
- [Prerequisites](#prerequisites)
  - [Developer Ad Network Application](#developer-ad-network-application)
  - [Support for Ad Network on user's Pi Browser](#support-for-ad-network-on-users-pi-browser)
- [Implementation](#implementation)
  - [Basic Usage](#basic-usage)
    - [Interstitial Ads](#interstitial-ads-1)
    - [Rewarded Ads](#rewarded-ads-1)
    - [Enabling Loading Banner Ads](#enabling-loading-banner-ads)
  - [Advanced Usage](#advanced-usage)
    - [Interstitial Ads Advanced Usage](#interstitial-ads-advanced-usage)
    - [Rewarded Ads Advanced Usage](#rewarded-ads-advanced-usage)
    - [Rewarded ads status verification with Pi Platform API](#rewarded-ads-status-verification-with-pi-platform-api)

## Introduction

Pi App Platform gives developers the opportunity to integrate ads into their Pi App and get rewarded. Displaying ads is open to all applications in the Pi ecosystem, but only applications approved by Pi Core Team can be monetized.
There are three types of ads supported currently: **interstitial**, **rewarded** and **banner**.

### Interstitial Ads

Interstitial ads are full-screen ads and are meant to be displayed between some kind of content, usually at natural transition points or breaks, such as in between activities or game levels.

Technically, interstitial ads can be displayed at any time, but check out our best practices documentation to learn more.

### Rewarded Ads

Rewarded ads are full-screen ads and are meant to be displayed to users in exchange for a reward within developer's app. For example if you're developing a game app and player runs out of lives but wants to continue playing, you might give him an option of obtaining extra live for watching a rewarded ad.

Since rewarded ads are meant to reward a user, they can be displayed only if user of an app is authenticated.

### Banner Ads

Banner ads are overlay ads that appear at the top or bottom of the screen while users are interacting with your app. They are less intrusive than full-screen ads and can be displayed continuously during app usage. Banner ads are ideal for maintaining a persistent ad presence without disrupting the user experience.

Since banner ads are always visible, they should be placed in non-intrusive locations that don't interfere with the main content or functionality of your app.

Currently banner ads **are not** supported via Pi SDK, but you can utilize them by enabling Loading Banner Ads setting in Ads Settings in Developer Portal. Loading Banner Ads are automatically displayed to users while your application is being loaded. To learn more, refer to [Enabling Loading Banner Ads](#enabling-loading-banner-ads) section.

## Prerequisites

In order to be rewarded from the Pi Developer Ad Network you must apply for it on the Pi Developer Portal. Once you've applied your application will be reviewed by the Pi Core Team.

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

#### Enabling Loading Banner Ads

You can enable Loading Banner Ads from Developer Portal. Go to `https://develop.pinet.com` in Pi Browser and find the application you'd like to enable Loading Banner Ads for. Go to **Dev Ad Network** menu item and select **Settings**. From there toggle the **Enable Loading Banner Ads** option. You can disable it anytime.

Your app's ads-related settings are cached so to see the effect you might need to wait or reset Pi Browser to invalidate the cache.

### Advanced Usage

While the examples from previous section are enough to have your application start displaying ads to your users, you might find yourself needing to perform additional actions like manually requesting ads or checking ads availability.

In some cases, loading ads may fail or be interrupted. If you detect that an ad is not available, you can try manually calling `Pi.Ads.requestAd()` passing ad type as an argument.

In other cases, you might need to know whether an ad is ready before calling `Pi.Ads.showAd()` method. You can check it using `Pi.Ads.isAdReady()` method passing ad type as an argument.

For the full definitions of Pi SDK methods with all possible response types, check the [SDK reference](SDK_reference.md#ads).

#### Interstitial Ads Advanced Usage

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

#### Rewarded Ads Advanced Usage

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
