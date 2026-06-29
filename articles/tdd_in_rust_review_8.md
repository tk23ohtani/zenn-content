---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 §8
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
- [ ] 通過の概念
```

DollarとFrancを消してしまいたいが、一気に消すにはステップが大きすぎるとのことで、小さく一歩ずつサブクラスへの三唱箇所を減らしていく作戦をとる。  
本ではまずは、MoneyにDollarを返すFactory Methodを定義しているが、

```rust:lib.rs
mod tests {
    use crate::Dollar;
    use crate::Franc;
    use crate::Money;   // 参照を追加
    use crate::MoneyTrait;

    #[test]
    fn test_multiplication() {
        let mut five = Money::dollar(5);
        assert_eq!(Dollar::new(10), five.times(2));
        assert_eq!(Dollar::new(15), five.times(3));
    }
    // ...
```

❌コンパイラが親切にMoneyにdollarメソッドが無いことを教えてくれる。

```rust:lib.rs
impl Money {
    // ...
    fn dollar(_amount: i32) -> Money {
        Money {
            amount: _amount,
            typeid: TypeId::of::<Dollar>(),
        }
    }
}
```

✅testがGreenになり、残りの箇所もFactory Methodを使うようテストを書き換える。

```rust:lib.rs
mod tests {
    use crate::Dollar;
    use crate::Franc;
    use crate::Money;
    use crate::MoneyTrait;

    #[test]
    fn test_multiplication() {
        let mut five = Money::dollar(5);
        assert_eq!(Money::dollar(10), five.times(2));
        assert_eq!(Money::dollar(15), five.times(3));
    }

    #[test]
    fn test_equality() {
        assert!(Money::dollar(5).equals(Money::dollar(5)));
        assert!(!Money::dollar(5).equals(Money::dollar(6)));
        assert!(Franc::new(5).equals(Franc::new(5)));
        assert!(!Franc::new(5).equals(Franc::new(6)));
        assert!(!Franc::new(5).equals(Dollar::new(5).to_money()));
    }
    // ...
}
```

✅素晴らしい、テストはGreenのままだ。  
Francも同様にFactory Methodを使うよう、テストを書き換える。

```rust:lib.rs
mod tests {
    // ...
    #[test]
    fn test_equality() {
        assert!(Money::dollar(5).equals(Money::dollar(5)));
        assert!(!Money::dollar(5).equals(Money::dollar(6)));
        assert!(Money::franc(5).equals(Money::franc(5)));
        assert!(!Money::franc(5).equals(Money::franc(6)));
        assert!(!Money::franc(5).equals(Money::dollar(5)));
    }

    #[test]
    fn test_franc_multiplication() {
        let mut five = Money::franc(5);
        assert_eq!(Money::franc(10), five.times(2));
        assert_eq!(Money::franc(15), five.times(3));
    }
}
```

❌コンパイラが親切にMoneyにdollarメソッドが無いことを教えてくれる。


```rust:lib.rs
impl Money {
    // ...
    fn franc(_amount: i32) -> Money {
        Money {
            amount: _amount,
            typeid: TypeId::of::<Franc>(),
        }
    }
}
```

✅テストがGreenになった。

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
- [ ] 通過の概念
- [ ] testFrancMultiplicationを削除する？
```
