---
title: "XRPL AMM"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## XRPL AMM

https://xrpl.org/ja/known-amendments.html#amm

## AMM

既存の分散型取引所と統合された形で、自動マーケットメーカー(AMM)機能に追加します。

各アセット（トークンまたはXRP）のペアは、Ledger上に最大1つのAMMプールを持つことができ、誰でも流動性を提供することで、収益と為替リスクを比例配分することができます。各AMMプールのインスタンスはその資産を保持するための特別なアカウントを持ち、流動性プロバイダーに対してその預入額に応じて"LPトークン"を発行します。流動性プロバイダーは、LPトークンのシェアに基づいてAMMプールの取引手数料に投票することができます。ユーザは、一定期間取引手数料が割引される権利にLPトークンを使って入札することができます。

追加される新規トランザクション:

- AMMBid - 取引手数料を割引するAMMプールのオークション枠に入札します。
- AMMCreate - AMMプールの新しいインスタンスを作成し、初期資金を供給します。
- AMMDelete - 空となったAMMプールのインスタンスを削除します。
- AMMDeposit - 既存のAMMプールに資金を供給し、LPトークンを受け取ります。
- AMMWithdraw - LPトークンをAMMプールに返却して資金を引き出します。
- AMMVote - AMMプールの取引手数料について投票します。

既存のトランザクションを新たな機能でアップデートします。

資産の取引に利用可能なPaymentとOfferCreateトランザクションは、より最良の取引レートを利用可能とするためにOfferとAMMの任意の組み合わせを利用します。
AMMの特別なアカウントに送信できないトランザクションがあります（例えば、AMMはCheckを換金できないため、AMMへのCheckCreateは許可されません）。
新しいタイプのレジャーエントリAMMを追加し、AccountRootレジャーエントリタイプにAMMIDフィールドを追加します。

いくつかの新しい結果コードを追加します。

## AMMBid

AMMのオークションスロットに入札します。

勝った場合は、入札額が高くなるか 24 時間が経過するまで、割引料金で AMM と取引できます。24 時間が経過する前に入札額が高かった場合は、残り時間に基づいて入札費用の一部が返金されます。AMM の LP トークンを使用して入札します。落札額がAMMに返還され、LPトークンの未払い残高が減少します。

Interface: https://js.xrpl.org/interfaces/AMMBid.html

## AMMCreate

資産のペアを取引するための新しいAMMインスタンスを作成します。

Interface: https://js.xrpl.org/interfaces/AMMCreate.html

## AMMDelete

https://js.xrpl.org/interfaces/AMMDelete.html

## AMMDeposit

https://js.xrpl.org/interfaces/AMMDeposit.html

## AMMWithdraw

https://js.xrpl.org/interfaces/AMMWithdraw.html

## AMMVote

https://js.xrpl.org/interfaces/AMMVote.html
