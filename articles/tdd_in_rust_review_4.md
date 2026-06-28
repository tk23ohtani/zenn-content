---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 #4
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
- [ ] amountのprivateにする
- [x] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
- [x] equals()
- [ ] hashCode()
- [ ] nullとの等価性比較
- [ ] 他のオブジェクトとの等価性比較
```

Dollar同士を直接比較してみよう。

```rust:lib.rs
#[cfg(test)]
mod tests {
    use crate::Dollar;

    #[test]
    fn test_multiplication() {
        let mut five = Dollar::new(5);
        let mut product = five.times(2);
        assert_eq!(Dollar::new(10), product);
        product = five.times(3);
        assert_eq!(15, product.amount);
    }

    #[test]
    fn test_equality() {
        assert!(Dollar::new(5).equals(Dollar::new(5)));
        assert!(!Dollar::new(5).equals(Dollar::new(6)));
    }
}
```

❌Rustでは比較演算子のオーバロードを使わないと比較できないので、`PartialEq`トレイトを使用する。

```rust:lib.rs
impl PartialEq for Dollar {
    fn eq(&self, _other: &Self) -> bool {
        self.amount == _other.amount
    }
}
```

オーバーロードにはDebug指定が必要。

```rust:lib.rs
#[derive(Debug)]
pub struct Dollar {
    amount: i32,
}
```

✅テストが通った。  
もう一つも書き換える。

```rust:lib.rs
#[cfg(test)]
mod tests {
    use crate::Dollar;

    #[test]
    fn test_multiplication() {
        let mut five = Dollar::new(5);
        let mut product = five.times(2);
        assert_eq!(Dollar::new(10), product);
        product = five.times(3);
        assert_eq!(Dollar::new(15), product);
    }

    #[test]
    fn test_equality() {
        assert!(Dollar::new(5).equals(Dollar::new(5)));
        assert!(!Dollar::new(5).equals(Dollar::new(6)));
    }
}
```

一次変数productは必要なくなったので。

```rust:lib.rs

#[cfg(test)]
mod tests {
    use crate::Dollar;

    #[test]
    fn test_multiplication() {
        let mut five = Dollar::new(5);
        // let mut product = five.times(2);
        assert_eq!(Dollar::new(10), five.times(2));
        // product = five.times(3);
        assert_eq!(Dollar::new(15), five.times(3));
    }

    #[test]
    fn test_equality() {
        assert!(Dollar::new(5).equals(Dollar::new(5)));
        assert!(!Dollar::new(5).equals(Dollar::new(6)));
    }
}
```

amountはもともとprivateだった。

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

