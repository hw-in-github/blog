---
title: Xcode Tips
date: 2017-10-20 14:09:22
tags:
---

How to get Xcode to stop generating copyright headers on your Swift files

<!--more-->

## Here’s how to get rid of the clutter and make your Swift files clean.

![](assets/Xcode-Tips-8447b3e0.png)

### Create a new User File Template

Quit Xcode. Bad things may happen if it’s open while you’re creating a new file template.

Launch Terminal.

```
mkdir -p ~/Library/Developer/Xcode/Templates/File Templates/User Templates
cd ~/Library/Developer/Xcode/Templates/File Templates/User Templates
cp -r /Applications/Xcode.app/Contents/Developer/Library/Xcode/Templates/File\ Templates/Source/Swift\ File.xctemplate Empty\ Swift\ File.xctemplate
```

Edit your file template
Open the Empty Swift `File.xctemplate/___FILEBASENAME___.swift` in your favorite editor.
Delete the copyright header.
Choose your file template when creating a new Swift file

Now it’s safe to launch Xcode. Go ahead and do that now. Now, whenever you create a new Swift file, do the following:

* Create a New File.
* Choose User Templates on the left.
* Choose your Empty Swift File.
* Empty Swift File

![](assets/Xcode-Tips-e31c518d.png)

And enjoy having a little less clutter in your life.
Become a better iOS developer with more of these tips and tricks by joining the mailing list. You’ll establish a foundation in Swift with the 5-Part Guide to Getting Started with Swift, and you’ll continue to get better at iOS with more articles just like this.
