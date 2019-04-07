---
title: pitfalls
date: 2017-11-01 22:37:22
tags:
---

UIButton不调用Target Selector
```swfit
var checkBtn: UIButton = {
        let check = UIButton()
        check.setImage(UIImage(named: "photo_check_selected"), for: .selected)
        check.setImage(UIImage(named: "photo_check_default"), for: .normal)
        check.addTarget(self, action: #selector(checkBtnTapped(sender:)), for: .touchUpInside)
        return check
    }()
func checkBtnTapped(sender: UIButton) {
        print("check button tapped")
        sender.isSelected = !sender.isSelected
    }
发现selector不被调用，`var`前面要加`lazy`，否则block里的self没初始化
```
