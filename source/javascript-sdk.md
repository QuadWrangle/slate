---
title: QuadWrangle JavaScript SDK

language_tabs:
	- javascript

toc_footers:
	- <a href="#">TOC</a>

includes:
	- errors

search: true
---

# Introduction
The QuadWrangle JavaScript SDK (or just "SDK") is a set of JavaScript functions you can use to interact with the QuadWrangle platform. This SDK provides functions that covers all functionality, from user authentication, profile saving, event queries, creating meetups, posting ClassRing requests, posting new ClassNotes etc. 

In addition to the SDK, there are special settings available to you in your instance of QuadHub that will let you define the "routes" (or URLs) within your QuadWeb site so that the various system-generated emails all have your own routing included (for example, the reset password email, that needs to know what your specific reset password URL will be)

# Usage

To use this SDK, simply include it in your HTML file (this SDK has been designed to not require any external 3rd party libraries and is not overly "opinionated" and lends itself well to SPAs "Single Page Applications" but does not require you to use Angular, or React or Vue. You may use any of those libraries with the SDK). 

## Production:
```html
<script src="https://sdk.qwd.io/<api_key>"></script>
```

## Sandbox
```html
<script src="http://sandboxsdk.qwd.io/<api_key>"></script>
```

# System Functions

System functions represent those functions that will aid  you in developing the foundation of your site. 

## Navigation

The QuadWeb platform provides for several varieties of navigation, which are: Header Navigation, Masthead Navigation, Main Menu, Footer Navigation and Basement navigation. 

Each of these navigation sections can be configured in QuadHub (Admin Tools > QuadWeb Options)



# Account Functions

# Public Profile Functions

# News Functions

# Events Functions

# Meetups Functions

# ClassRing Functions

# ClassNotes Functions

# Directory Functions

# Giving Page Functions

# Photo Gallery Functions

# Homepage Functions

# Pages Functions

# Site Search