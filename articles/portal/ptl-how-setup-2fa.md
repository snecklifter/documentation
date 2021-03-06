---
title: How to set up two-factor authentication for a user account
description: Describes how to set up two-factor authentication (2FA) for a user account
services: portal
author: Sue Highmoor
reviewer: wllewellyn
lastreviewed: 07/02/2020

toc_rootlink: How To
toc_sub1:
toc_sub2:
toc_sub3:
toc_sub4:
toc_title: Set up 2FA for a user account
toc_fullpath: How To/ptl-how-setup-2fa.md
toc_mdlink: ptl-how-setup-2fa.md
---

# How to set up two-factor authentication for a user account

## Overview

If your Portal administrator has enabled two-factor authentication (2FA) in the UKCloud Portal, you must enter a randomly generated code, as well as your user name and password, when logging in to the Portal.

![Two-Factor Authentication dialog box in the UKCloud Portal](images/ptl-2fa.png)

The first time you log into the Portal after 2FA is enabled, you'll need to to set up 2FA for your user account by registering your account with an authenticator app, such as Google Authenticator.

> [!NOTE]
> If at any point you need to reset 2FA on your user account, for example if you lose or replace your phone, contact your Portal administrator. After your administrator resets your 2FA, the next time you log in to the Portal, you'll need to set up 2FA again.

### Intended audience

This article is intended for any Portal user who needs to set up 2FA to log in to the Portal.

## Setting up 2FA

When you first log in to a Portal account that has 2FA enabled, you'll be prompted to set up 2FA:

1. Log in to the UKCloud Portal, providing your user name and password as usual.

    For more detailed instructions, see the [*Getting Started Guide for the UKCloud Portal*](ptl-gs.md).

2. Install an authenticator app if you don't already have one which is compatible with the UKCloud Portal. We've tested 2FA with Google Authenticator, which is a smart phone app, and Authy, which is available as a smart phone app, desktop app or Chrome plug-in. Other authenticator apps may also be compatible but have not been tested. Once you've installed the authenticator app, scan the QR code or copy the security key.

    This is a one-off set up and does not need to be repeated as part of the normal Portal log-in process.

    ![Configure your two-factor authentication dialog box](images/ptl-2fa-setup.png)

3. When you've installed and set up your authenticator app, enter the 6 digit code provided by the app in the *Configure your two-factor authentication* dialog box and click **Confirm code** to log in to the Portal.

4. For future logins to the Portal, use your authenticator app to generate a 2FA code to confirm your identity and complete the log in process.

## Related videos

- [*UKCloud Portal overview video*](ptl-vid-portal.md)

## Feedback

If you find a problem with this article, click **Improve this Doc** to make the change yourself or raise an [issue](https://github.com/UKCloud/documentation/issues) in GitHub. If you have an idea for how we could improve any of our services, send an email to <feedback@ukcloud.com>.
