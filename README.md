<img src="docs/banner.png" align="center" title="Trust logo">

Trust Wallet Core is an open-source, cross-platform, mobile-focused library
implementing low-level cryptographic wallet functionality for a high number of blockchains.
It is a core part of the popular [Trust Wallet](https://trustwallet.com), and some other projects.
Most of the code is C++ with a set of strict C interfaces, and idiomatic interfaces for supported languages:
Swift for iOS and Java (Kotlin) for Android.

[![iOS CI](https://github.com/trustwallet/wallet-core/actions/workflows/ios-ci.yml/badge.svg)](https://github.com/trustwallet/wallet-core/actions/workflows/ios-ci.yml)
[![Android CI](https://github.com/trustwallet/wallet-core/actions/workflows/android-ci.yml/badge.svg)](https://github.com/trustwallet/wallet-core/actions/workflows/android-ci.yml)
[![Linux CI](https://github.com/trustwallet/wallet-core/actions/workflows/linux-ci.yml/badge.svg)](https://github.com/trustwallet/wallet-core/actions/workflows/linux-ci.yml)
[![Rust CI](https://github.com/trustwallet/wallet-core/actions/workflows/linux-ci-rust.yml/badge.svg)](https://github.com/trustwallet/wallet-core/actions/workflows/linux-ci-rust.yml)
[![Wasm CI](https://github.com/trustwallet/wallet-core/actions/workflows/wasm-ci.yml/badge.svg)](https://github.com/trustwallet/wallet-core/actions/workflows/wasm-ci.yml)
[![Kotlin CI](https://github.com/trustwallet/wallet-core/actions/workflows/kotlin-ci.yml/badge.svg)](https://github.com/trustwallet/wallet-core/actions/workflows/kotlin-ci.yml)
[![Docker CI](https://github.com/trustwallet/wallet-core/actions/workflows/docker.yml/badge.svg)](https://github.com/trustwallet/wallet-core/actions/workflows/docker.yml)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=TrustWallet_wallet-core&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=TrustWallet_wallet-core)

[![Gitpod Ready-to-Code](https://img.shields.io/badge/Gitpod-ready--to--code-blue?logo=gitpod)](https://gitpod.io/#https://github.com/trustwallet/wallet-core)
![GitHub](https://img.shields.io/github/license/TrustWallet/wallet-core.svg)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/trustwallet/wallet-core)
![SPM](https://img.shields.io/badge/SPM-ready-blue)
![Cocoapods](https://img.shields.io/cocoapods/v/TrustWalletCore.svg)

# AeonCoreX-Lab Fork

## Purpose

This repository is a fork of [trustwallet/wallet-core](https://github.com/trustwallet/wallet-core), maintained by [AeonCoreX-Lab](https://github.com/AeonCoreX-Lab) as the wallet-core dependency for [AeonVault](https://github.com/AeonCoreX-Lab/AeonVault).

Vendoring upstream source directly into AeonVault would mean manually re-applying any customizations on every upstream update, with no clean way to tell "our changes" apart from "upstream changes." Forking instead gives AeonVault:

- A stable, versioned Android artifact (AAR) to depend on like any other Maven library, instead of building wallet-core from source inside the app's own build.
- A place to carry AeonVault-specific customizations on top of upstream, isolated on their own branch, without losing the ability to pull in upstream fixes (in particular security patches) on an ongoing basis.
- An explicit, human-reviewed gate on every upstream sync — nothing merges into the branch AeonVault actually builds from without a clean fast-forward or a reviewed PR.

## What this fork does

1. **Mirrors upstream** — `main` tracks `trustwallet/wallet-core:master` exactly, kept in sync automatically.
2. **Layers AeonVault's customizations on top** — `aeoncorex-custom` is `main` plus AeonVault-specific changes, and is the branch everything is actually built from.
3. **Builds and publishes a versioned Android AAR** — on a relevant change (or manual trigger), CI builds `aeoncorex-custom` and publishes the release AAR to this repo's own GitHub Packages Maven registry under an explicit version string.
4. **Keeps upstream syncs safe** — a clean merge from `main` is pushed automatically; anything that conflicts with AeonVault's customizations (most likely in signing/key-derivation/address code) stops and opens a PR instead of being auto-resolved.

**Branch layout**

| Branch | Purpose |
| --- | --- |
| `main` | Pure mirror of upstream `trustwallet/wallet-core:master`. Fast-forward only — no direct commits. |
| `aeoncorex-custom` | AeonVault's customizations, merged on top of `main`. All app-facing builds are published from here. |

**Automation**

- [`sync-upstream.yml`](.github/workflows/sync-upstream.yml) — runs weekly (Mondays 03:00 UTC) and on manual dispatch. Fast-forwards `main` from upstream, then merges `main` into `aeoncorex-custom`. Clean merges are pushed automatically; conflicts open a PR against `aeoncorex-custom` for manual review instead of being auto-resolved.
- [`build-android-aar.yml`](.github/workflows/build-android-aar.yml) — builds `aeoncorex-custom` on relevant changes (or manual dispatch with an explicit version) and publishes the release AAR to this repo's GitHub Packages Maven registry.

## Required secrets / tokens

| Name | Used by | Where it comes from |
| --- | --- | --- |
| `GITHUB_TOKEN` | Both workflows (sync + publish) | Auto-provided by GitHub Actions for every run — nothing to create. Its *permissions* still need `packages: write` (see below), granted either by the workflow's own `permissions:` block or, if that's not enough, by Settings → Actions → General → Workflow permissions → "Read and write permissions". |
| `GPR_USER` / `GPR_TOKEN` | AeonVault (consumer side, not this repo) | A **Personal Access Token** — Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token, scope `read:packages`. `GITHUB_TOKEN` only works within its own repo's run; reading a package from a different repo always needs a real PAT, even within the same org. Store locally in `~/.gradle/gradle.properties` (never commit it) or as a repo secret under AeonVault's Settings → Secrets and variables → Actions. |

No other secrets are required to publish from this repo — `sync-upstream.yml` and `build-android-aar.yml` both run entirely on the default `GITHUB_TOKEN`.

## Using the published AAR

AeonVault (or any other consumer) pulls this as a normal Gradle dependency instead of vendoring source:

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        maven {
            name = "AeonCoreXWalletCore"
            url = uri("https://maven.pkg.github.com/AeonCoreX-Lab/wallet-core")
            credentials {
                username = System.getenv("GPR_USER")
                password = System.getenv("GPR_TOKEN")
            }
        }
    }
}
```

```kotlin
// app/build.gradle.kts
implementation("com.trustwallet:wallet-core:<published-version>")
```

Always pin to the exact version string printed in the `build-android-aar.yml` run summary — floating versions are intentionally not supported, since silent upgrades to signing/key-derivation code are not acceptable here.

# Documentation

For comprehensive documentation, see [developer.trustwallet.com](https://developer.trustwallet.com/wallet-core).

# Audit Reports

Security Audit reports can be found in the [audit](audit) directory.

# Supported Blockchains

Wallet Core supports more than **130** blockchains: Bitcoin, Ethereum, BNB, Cosmos, Solana, and most major blockchain platforms.
The full list is [here](docs/registry.md).

# Building

For build instructions, see [developer.trustwallet.com/wallet-core/building](https://developer.trustwallet.com/wallet-core/building).


# Using from your project

If you want to use wallet core in your project follow these instructions.

## Android

Android releases are hosted on [GitHub packages](https://github.com/trustwallet/wallet-core/packages/700258), you need to add GitHub access token to install it. Please check out [this installation guide](https://developer.trustwallet.com/wallet-core/integration-guide/android-guide#adding-library-dependency) or `build.gradle` from our [android sample](https://github.com/trustwallet/wallet-core/blob/master/samples/android/build.gradle)

Don't forget replacing the version in the code with latest: ![GitHub release (latest by date)](https://img.shields.io/github/v/release/trustwallet/wallet-core)

## iOS

We currently support Swift Package Manager and CocoaPods (will discontinue in the future).

### SPM

Download latest `Package.swift` from [GitHub Releases](https://github.com/trustwallet/wallet-core/releases) and put it in a local `WalletCore` folder.

Add this line to the `dependencies` parameter in your `Package.swift`:

```swift
.package(name: "WalletCore", path: "../WalletCore"),
```

Or add remote url + `master` branch, it points to recent (not always latest) binary release.

```swift
.package(name: "WalletCore", url: "https://github.com/trustwallet/wallet-core", .branchItem("master")),
```

Then add libraries to target's `dependencies`:

```swift
.product(name: "WalletCore", package: "WalletCore"),
.product(name: "WalletCoreSwiftProtobuf", package: "WalletCore"),
```

### CocoaPods

Add this line to your Podfile and run `pod install`:

```ruby
pod 'TrustWalletCore'
```

## NPM (beta)

```js
npm install @trustwallet/wallet-core
```

## Go (beta)

Please check out the [Go integration sample](https://github.com/trustwallet/wallet-core/tree/master/samples/go).

## Kotlin Multipleplatform (beta)

Please check out the [Kotlin Multiplatform sample](https://github.com/trustwallet/wallet-core/tree/master/samples/kmp)

# Projects

Projects using Trust Wallet Core. Add yours too!

[<img src="https://trustwallet.com/icon.svg" alt="Trust Wallet"/>](https://trustwallet.com)

[Coinpaprika](https://coinpaprika.com/)
| [crypto.com](https://crypto.com)
| [Frontier](https://frontier.xyz/)
| [Tokenary](https://tokenary.io/)
| [MemesWallet](https://planetmemes.com/)
| [xPortal](https://xportal.com/)
| [Slingshot](https://slingshot.finance/)
| [ECOIN Wallet](https://play.google.com/store/apps/details?id=org.ecoinwallet&pcampaignid=web_share)

# Community

There are a few community-maintained projects that extend Wallet Core to some additional platforms and languages. Note this is not an endorsement, please do your own research before using them:

- Flutter binding https://github.com/weishirongzhen/flutter_trust_wallet_core
- Python binding https://github.com/phuang/wallet-core-python
- Wallet Core on Windows https://github.com/kaetemi/wallet-core-windows

# Contributing

The best way to submit feedback and report bugs related to WalletCore is to [open a GitHub issue](https://github.com/trustwallet/wallet-core/issues/new).
If the bug is not related to WalletCore but to the TrustWallet app, please [create a Customer Support ticket](https://support.trustwallet.com/en/support/tickets/new).
If you want to contribute code please see [Contributing](https://developer.trustwallet.com/wallet-core/contributing).
If you want to add support for a new blockchain also see [Adding Support for a New Blockchain](https://developer.trustwallet.com/wallet-core/newblockchain), make sure you have read the [requirements](https://developer.trustwallet.com/wallet-core/newblockchain#requirements) section.

Thanks to all the people who contribute.
<a href="https://github.com/trustwallet/wallet-core/graphs/contributors"><img src="https://opencollective.com/wallet-core/contributors.svg?width=890&button=false" /></a>

# Disclaimer

The Wallet Core project is led and managed by Trust Wallet with a large contributor community and actively used in several projects.  Our goal at Wallet Core is to give other wallets an easy way to add chain support.

Trust Wallet products leverage wallet core, however, they may or may not leverage all the capabilities, features, and assets available in wallet core due to their own product requirements.

# License

Trust Wallet Core is available under the Apache 2.0 license. See the [LICENSE](LICENSE) file for more info.
