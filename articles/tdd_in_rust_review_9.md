---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 §9
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
- [ ] testFrancMultiplicationを削除する？
```

ここで一度、不要な実装を全て駆除しておく。
テストについては不要な宣言を削除するのみとする。

```rust:lib.rs
mod tests {
    // use crate::Dollar;
    // use crate::Franc;
    use crate::Money;
    // use crate::MoneyTrait;

    // ...
} 
```

`MoneyTrait`は不要になっているはずだ。

```rust:lib.rs
// trait MoneyTrait {
//     fn new(_amount: i32) -> Money;
// }
```

```rust:lib.rs
// impl MoneyTrait for Dollar {
//     fn new(_amount: i32) -> Money {
//         Money {
//             amount: _amount,
//             typeid: TypeId::of::<Dollar>(),
//         }
//     }
// }
```

```rust:lib.rs
// impl MoneyTrait for Franc {
//     fn new(_amount: i32) -> Money {
//         Money {
//             amount: _amount,
//             typeid: TypeId::of::<Franc>(),
//         }
//     }
// }
```

✅テストはGreenのままのはずだ。  
本に従って通過を実装するためのテストを書く。文字列からだ。
だが、Rustの制約により、本よりも一般化が進んでいるので、その進みに合わせたものとする。
テストは本の通りで良さそうだ。

```rust:lib.rs
    #[test]
    fn test_currency() {
        assert_eq!("USD", Money::dollar(1).currency());
        assert_eq!("CHF", Money::franc(1).currency());
    }
```

❌コンパイラが親切にMoneyにcurrencyメソッドが無いことを教えてくれる。
とりあえず、ビルドを通るようにしてみよう。

```rust:lib.rs
impl Money {
    // ...
    fn currency(&self) -> String {
        String::from("JPY")
    }
}
```

❌当然テストはRedだ。
本では継承関係を使ってDollarクラスとFrancクラスに抽象関数を定義しているが、Rustでは難しい。そこで、本のP.58まで飛んで、Moneyに通貨の文字列を持つようにしてみよう。

```rust:lib.rs
pub struct Money {
    amount: i32,
    typeid: TypeId,
    currency: &'static str,
}
```

❌とりあえず、全て”JPY”で返すようにしてビルドを通してみよう。

```rust:lib.rs
impl Money {
    fn equals(&self, _other: Money) -> bool {
        self.amount == _other.amount && self.typeid == _other.typeid
    }
    fn times(&mut self, _multiplier: i32) -> Money {
        Money {
            amount: self.amount * _multiplier,
            typeid: self.typeid,
            currency: "JPY",
        }
    }
    fn dollar(_amount: i32) -> Money {
        Money {
            amount: _amount,
            typeid: TypeId::of::<Dollar>(),
            currency: "JPY",
        }
    }
    fn franc(_amount: i32) -> Money {
        Money {
            amount: _amount,
            typeid: TypeId::of::<Franc>(),
            currency: "JPY",
        }
    }
    fn currency(&self) -> &'static str {
        "JPY"
    }
}
```

✅currencyメソッドが”USD”を返すようにすれば、dollar側のテストはパスする。

```rust:lib.rs
    fn currency(&self) -> String {
        currency: "USD",
    }
```

✅このまま、dollar側の実装を進めよう。
greenを明らかにするために、一旦、franc側のテストをコメントアウトし、TODOリストに復元するよう課題を書き足しておく。これでテストがGreenになる。このGrennを保つよう、Dollar側の実装を進めてみよう。  
まずは、dollarメソッドで”USD”を埋め込む。

```rust:lib.rs
    fn dollar(_amount: i32) -> Money {
        Money {
            amount: _amount,
            typeid: TypeId::of::<Dollar>(),
            currency: "USD",
        }
    }
```

✅currencyメソッドでMoneyのフィールドを返してみる。

```rust:lib.rs
    fn currency(&self) -> &'static str {
        self.currency
    }
```

✅素晴らしい、Greenが保たれている。  
では、Francのテストを復元して、Franc側にも対応しよう。

```rust:lib.rs
    #[test]
    fn test_currency() {
        assert_eq!("USD", Money::dollar(1).currency());
        assert_eq!("CHF", Money::franc(1).currency());
    }
```

❌まずはRedを確認した。  
francメソッドの通貨文字列を”CHF”に変えてみよう。

```rust:lib.rs
    fn franc(_amount: i32) -> Money {
        Money {
            amount: _amount,
            typeid: TypeId::of::<Franc>(),
            currency: "CHF",
        }
    }
```

✅素晴らしい、Greenに戻すことが出来た。  
本のような歩幅ではなかったが、Rustらしい進み方だろうと私は思う。

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
