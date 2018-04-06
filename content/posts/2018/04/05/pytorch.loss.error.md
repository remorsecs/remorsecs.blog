---
title: "PyTorch loss error: don't compute the gradient w.r.t. targets..."
date: 2018-04-05T21:25:48+08:00
categories: ['PyTorch']
tags: ['PyTorch', 'Deep Learning', 'loss', 'debug']

---

剛剛寫了一段程式碼，在下面這一段出現錯誤：

```Python
criterion = nn.MSELoss()
loss = criterion(target, outputs)
```

錯誤訊息：

> AssertionError: nn criterions don't compute the gradient w.r.t. targets - please mark these variables as volatile or not requiring gradients

看半天看不出錯在哪裡，也不是 model 裡面要設定 `parameter.requires_grad = True` 的問題。結果在 PyTorch 官網翻半天，發現有人遇到一樣的問題，解法是：

```Python
loss = criterion(outputs, target)
```

原來是放反了......在 PyTorch 裡面，ground truth 應該要放在右邊，避免去計算無效的 gradient 。但特別要注意的是，在其他框架裡面不一定都是放在右邊，像是 Keras 或是 scikit-learn 是把 ground truth 放在左邊的。

不過在 MSE loss 內放反不會有問題的才對，只是求值的話，不管放哪邊計算結果都一樣。PyTorch 會對 Variable 計算 gradient，以上面的案例來說就是會去算了一個用不到的值。但用不到就是沒用而已，也不該引發錯誤直接終止程式的才對。

於是翻了 source code 看了一下，果然這一段是用 `assert` 引發的（其實錯誤訊息開頭就講了 Orz 下次要仔細理解錯誤訊息啊）。因為是由 `assert` 引發，所以只要在命令列參數加入 -O 就會把 `__debug__` 設定為 False，如此一來就不會因為這個問題報錯了。

不過更重要的教訓是要仔細查看官網的文件。

