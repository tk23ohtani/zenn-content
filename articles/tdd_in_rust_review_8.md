---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 #6
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
        let mut five = Money::new(5);
        assert_eq!(Dollar::new(10), five.times(2));
        assert_eq!(Dollar::new(15), five.times(3));
    }
    // ...
```

❌コンパイラが親切にMoneyにnewメソッドが無いことを教えてくれる。

```rust:lib.rs
impl Money {
    fn new(_amount: i32) -> Money {
        Money {
            amount: _amount,
            typeid: TypeId::of::<Money>(),
        }
    }
    // ...
}
```

✅testがGreenになり、

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


インスタンス化した時の型を比較することにする。

```rust:lib.rs
```

なので、インスタンス化した時に型を持たせる。

```rust:lib.rs
}
```

❌ビルドエラー？！  
timesメソッドでもインスタンス化しているな。

```rust:lib.rs
impl Money {
    fn times(&mut self, _multiplier: i32) -> Money {
        Money {
            amount: self.amount * _multiplier,
            typeid: self.typeid
        }
    }
    // ...
}
```

✅testがGreenになり、FrancとDollarの比較が等しくならないようになった。もちろんレートが1:1ではない時ではある。


```rust:lib.rs
```

```rust:lib.rs
```












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
```



```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

✅testはGreenのままだ。  

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

