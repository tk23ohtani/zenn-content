---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 §11
emoji: ⚙
type: tech
topics:
  - TDD
  - Rust
published: false
---

TODO Listsを確認しておこう。

```markdown
- [ ] $5 + 10 CHF = $10
- [x] $5 * 2 = $10
- [x] amountのprivateにする
- [x] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
- [x] equals()
- [ ] hashCode()
- [ ] nullとの等価性比較
- [ ] 他のオブジェクトとの等価性比較
- [x] 5 CHF * 2 = 10 CHF
- [ ] DollarとFrancの重複
- [x] equalsの一般化
- [x] timesの一般化
- [x] FrancとDollarを比較する
- [x] 通過の概念
- [ ] testFrancMultiplicationを削除する？
- [x] Fnancの通貨判定テストを復元する
```

不要になったら消す章だが、DollarもFrancも参照されない構造体が残っているだけなので、そのまま消して良さそうだ。

```rust:lib.rs
// pub struct Dollar {}
// ...
// pub struct Franc {}
```

✅当然テストはGreenのままだ。  
Francの等価性比較は十分にされているから消すらしい。

```rust:lib.rs
mod tests {
    // ...
    fn test_equality() {
        assert!(Money::dollar(5).equals(Money::dollar(5)));
        assert!(!Money::dollar(5).equals(Money::dollar(6)));
        assert!(Money::franc(5).equals(Money::franc(5)));
        // assert!(!Money::franc(5).equals(Money::franc(6)));
        // assert!(!Money::franc(5).equals(Money::dollar(5)));
        assert_ne!(Money::franc(5), Money::dollar(5));
    }
    // ...
}
```

✅当然テストはGreenのままだ。  

```rust:lib.rs
mod tests {
    // ...
    // #[test]
    // fn test_franc_multiplication() {
    //     let mut five = Money::franc(5);
    //     assert_eq!(Money::franc(10), five.times(2));
    //     assert_eq!(Money::franc(15), five.times(3));
    // }
    // ...
}
```

✅当然テストはGreenのままだ。  
TODOに変更しておこう。

```markdown
- [ ] $5 + 10 CHF = $10
- [x] $5 * 2 = $10
- [x] amountのprivateにする
- [x] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
- [x] equals()
- [ ] hashCode()
- [ ] nullとの等価性比較
- [ ] 他のオブジェクトとの等価性比較
- [x] 5 CHF * 2 = 10 CHF
- [x] DollarとFrancの重複
- [x] equalsの一般化
- [x] timesの一般化
- [x] FrancとDollarを比較する
- [x] 通過の概念
- [x] testFrancMultiplicationを削除する？
- [x] Fnancの通貨判定テストを復元する
```
