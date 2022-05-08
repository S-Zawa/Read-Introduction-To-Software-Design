---
theme: seriph # https://sli.dev/themes/gallery.html
title: SlidevのMarkdown記法サンプル
download: false
lineNumbers: true
background: https://source.unsplash.com/collection/94734566/1920x1080
class: 'text-center'
---

# 「良いコード/悪いコードで学ぶ設計入門」<br>を読んで

366394 Shota.Aizawa

---
layout: intro

---

# 0章

はじめに

---

## 0. はじめに

### きっかけ
- GW中に「良いコード／悪いコードで学ぶ設計入門」を読みました
- まだ途中ですが順に皆さんに内容をかいつまんで展開したいと思います

- https://note.com/minodriven/n/n12af8005899f
- <https://www.amazon.co.jp/dp/4297127830?tag=note0e2a-22&linkCode=ogi&th=1&psc=1>

### 対象読者
- オブジェクト指向プログラミング言語の基礎知識はあるものの、設計がよくわからない/自信がない方、これから設計をしっかり学び始めようと考えている方

---

<v-click>
突然ですが、正方形の定義ってわかりますか？
</v-click>
<div v-click class="text-xl p-2">
四辺の長さがすべて等しい
</div>
<div v-click class="text-xl p-2">
内角がすべて直角
</div>

<!--
これらの正方形の定義を知っているため一致しない図形を見た際に正方形でないと知覚できます
-->

---


SW開発で以下の経験はあるあるではないでしょうか？
- どこかのコードを変更すると、別の箇所でバグが発生した
- 変更の影響があるそうな箇所をあちらこちら探し回らなければならなくなった
- コードを読んでいるだけで日が暮れてしまった
- 簡単だと思っていた仕様変更やバグ修正に何日も費やしてしまった

<div v-click class="text-xl p-2">
これらの原因がわからない理由は正方形のときと違い変更に強いあるべき構造を知らないためです
</div>
<div v-click class="text-xl p-2">
SWの成長を阻害する設計や実装上の問題を「悪魔」と例え、正体を知覚し正しい対処ができるようになることを目的とした内容となっています
</div>

---
layout: intro
---

# 1章

悪しき構造の弊害を知覚する

---
設計を蔑ろにすると起こる弊害
- コードを読み解くのに時間がかかる
- バグを埋め込みやすくなる
- 悪しき構造が更に悪しき構造を誘発する
---

## 1.1. 意味不明な命名

### 技術駆動命名

```csharp
class MemoryStateManager
{
    void ChangeIntValue01(int changeValue)
    {
        IntValue01 -= changeValue;
        if (IntValue01 < 0)
        {
            IntValue01 = 0;
            UpdateState02Flag();
        }
    }
}
```

- 型名を表すInt、メモリ制御を表すMemoryやFlagなど、プログラミング、コンピュータ用語に基づいた技術ベースでの命名を技術駆動命名と呼びます

---

### 連番命名

```csharp
class Class01
{
    void Method001();

    void Method002();

    void Method003();
}
```
- クラスやメソッドに対して番号付けで命名することを連番命名と呼びます

---

- 技術駆動命名、連番命名は意図が全く読み取れない悪しき手法

- 意図や目的を表現した命名をすることで構造が簡明になります

---

## 1.2. 理解を困難にする条件分岐のネスト

### 何重にもネストしたロジック

```csharp
    // 生存判定
    if(0 < member.HitPoint){
        // 行動可能判定
        if(member.CanAct()){
            // MP残存判定
            if(magic.ConstMagicPoint <= member.ConstMagicPoint){
                member.ConsumeMagicPoint(magic.ConstMagicPoint);
                member.Chant(magic);
            }
        }
    }
```

- 上記はRPGにおける魔法発動までの条件を実装した例です
- ネストしているとコードの見通しが悪くなりどこまでがif文の処理ロジックなのか読み解きのが難しくなります

---

## 1.3. さまざまな悪魔を招きやすいデータクラス



- データクラスは単純な構造でありながら様々な悪魔を招きやすいです
- 業務契約を扱うサービスにて契約金額を扱う仕様を例にします

### データしか持たないありがちなクラス構造

```csharp
public class ContractAmount
{
    public int AmountIncludingTax;  // 税込み金額
    public decimal SalesTaxRate;    // 消費税率
}
```
- 税込み金額と消費税率をpublicなインスタンス変数として持ち、自由にデータの出し入れが可能な構造です
- データの入れ物だけでなく税込み金額を計算するロジックが当然必要になります
---

### ContractManagerに書かれる金額計算ロジック

```csharp
public class ContractManager
{
    public ContractAmount ContractAmount;

    // 税込金額を計算する
    public int
    CalculateAmountIncludingTax(int amountExcludingTax, decimal salesTaxRate)
    {
        decimal multiplier = Decimal.Add(salesTaxRate, new decimal(1.0));
        decimal amountIncludingTax =
            Decimal.Multiply(multiplier, new decimal(amountExcludingTax));

        return decimal.ToInt32(amountIncludingTax);
    }

    // 契約を締結する
    public void Conclude()
    {
        int amountIncludingTax =
            CalculateAmountIncludingTax(amountExcludingTax, salesTaxRate);
        ContractAmount = new ContractAmount();
        ContractAmount.AmountIncludingTax = amountExcludingTax;
        ContractAmount.SalesTaxRate = salesTaxRate;
        // 省略
    }
}

```

---

### 1.3.1. 仕様変更時に牙をむく悪魔

- 先程のは別のクラスに税込金額の計算ロジックが実装されているパターンです
- このサービスにおいて消費税関係で仕様変更により、消費税の税率ロジックを変更しました
  - しかし、数日後に新しい消費税率になっていないとの障害報告があがりました
  - 同様のロジックが複数箇所にあるのではと疑い、調査をしたところ数十箇所で実装されていることが発覚しました

![](/img/class-diagram.png)

<!--
//www.plantuml.com/plantuml/png/SoWkIImgAStDuKhEIImkLd3EpoifIamkSSpDByqhgUPIKCZCAr60y3ppadDAKpBpqaCIAu0IAQd9cRc9EGh1YJcfnK2XeCIILAvQBZI3V1Fp4jDJYq0cEIVc99Vc05LX6gS1iYKHBEnQWH3MWLjIZ6I99iYiVB3kL0rDe9KmEtrJYuk1bj71IfYDZnldFcvSzsJtjATBPZrkx0Uo7pUjVzoyyd7JJjfQZnfF2ZOrkhheGOvD3Io8y2Z1voSkXzIy561Y0G00
-->

---

なぜ同様の計算ロジックが複数箇所に存在するのでしょうか？

<div v-click class="text-xl p-2">
税込み計算は金額を扱うあらゆるユースケースで必要になるため多くの箇所で実装されやすい
</div>

<div v-click class="text-xl p-2">
多くの場所で使うといいつつも、税込み計算ロジックをどこか一箇所にすればいいと考える方もいるでしょう
</div>
<div v-click class="text-xl p-2">
そうなると設計に無頓着だと税込み計算ロジックが実装されていることに気づかずに再実装されていしまう可能性があります
</div>

---

- これらの事態はデータを保持するクラスとデータを使って計算するロジックが離れているときに頻発します
- 離れているがゆえに複数実装されていても認知されず、再実装されてしまいます

<div v-click class="text-xl p-2">
関連するデータやロジック同士が分離し、バラバラになっているのを低凝集と言います<br>
低凝集により引き起こされる弊害を次ページ以降にまとめます
</div>

---

### 1.3.2. 重複コード

- 先程の例で示した通り、実装済みの機能があるのに未実装と勘違いし、同じようなロジックをいたるところに複数実装してしまう可能性が高まります
- 意図せず重複コードが量産されることになります

### 1.3.3. 修正漏れ

- 重複コードが多く実装されている場合、仕様変更時にすべての重複コードを変更しなければならず、修正漏れが生じバグとなります

### 1.3.4. 可読性低下
- 関連するコード同士が分散していると重複コードも含めすべてを探し出すのに膨大な時間が必要となります

---

### 1.3.5. 生焼けオブジェクト

```csharp

        ContractAmount amount = new ContractAmount();
        Console.WriteLine(amount.SalesTaxRate.ToString());

```
- 上記のコードを実行するとヌルポが発生します
- ContractAmountが初期化の必要なクラスであることを利用者側が知らないとバグが生じてしまう不完全なクラス（生焼けオブジェクト）です


---

### 1.3.6. 不正値の混入

不正とは仕様として正しくない状態を指します
- 注文数がマイナスになっている
- ゲームにおいてHPの値が最大値を超えてしまっている


### 不正値を混入可能
- 負数の消費税率を代入するなどデータクラスは不整地を与えることが容易にできてしまいます

```csharp

        ContractAmount amount = new ContractAmount();
        amount.SalesTaxRate = new decimal(-0.1);

```
- 不正値が混入しないようにデータクラスの利用側でバリデーションロジックを実装することがありますが、こちらも税込み計算ロジック同様に重複コードの発生原因となります

---

### 今までの話をまとめてできたContractAmountクラス


```csharp
    public class ContractAmount
    {
        public int AmountIncludingTax { get; }  // 税込み金額

        public int AmountExcludingTax { get; }  // 税抜き金額
        public decimal SalesTaxRate { get; }    // 消費税率

        public ContractAmount(int amountExcludingTax, decimal salesTaxRate)
        {
            if (salesTaxRate < 0)
            {
                throw new ArgumentException();
            }

            if (amountExcludingTax < 0)
            {
                throw new ArgumentException();
            }
            AmountExcludingTax = amountExcludingTax;
            SalesTaxRate = salesTaxRate;
            AmountIncludingTax = CalculateAmountIncludingTax(amountExcludingTax, salesTaxRate);
        }

        // 税込金額を計算する
        private int CalculateAmountIncludingTax(int amountExcludingTax, decimal salesTaxRate)
        {
            decimal multiplier = decimal.Add(salesTaxRate, new decimal(1.0));
            decimal amountIncludingTax = decimal.Multiply(multiplier, new decimal(amountExcludingTax));

            return decimal.ToInt32(amountIncludingTax);
        }
    }

```