---
title: Xcode template
date: 2017-10-23 23:09:01
tags:
---

# Collection Cell Initializers
```swift
override func prepareForReuse() {
    super.prepareForReuse()
}

override init(frame: CGRect) {
    super.init(frame: frame)
    commonInit()
}

required init?(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)
    commonInit()
}
```

# UIView Initializers
```swift
override init(frame: CGRect) {
        super.init(frame: frame)
        setupSubviews()
    }

    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        setupSubviews()
    }
```

# UIViewController Initializers
```swift
required init(post: Post) {
        self.post = post
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder aDecoder: NSCoder) {
        preconditionFailure("Post View Controller must be initialized by code")
    }
```
