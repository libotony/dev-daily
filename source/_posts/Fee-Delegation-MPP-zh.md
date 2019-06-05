---
title: Fee Delegation - MPP
date: 2019-05-25 17:19:52
tags: [dapp,mpp,fee delegation,vip191,多方支付,代付,交易费]
---

在开始之前，我们先描述几个概念：

`TxOrigin`: 交易发起方，通常是一个账户
`GasPayer`: 支付费用的账户
`Fee Delegation`: 交易费用委托(代付), 一般指交易费用不是由`TxOrigin`支付的场景
`Multi Party Payment`: 多方支付, `Fee Delegation` 的一种形式

## 起源

我们之前有一段在以太坊上面开发的经验，我们发现向大众推广区块链应有会有一个核心问题，用户对于`需要ETH来为应用/代币支付手续费`这件事情的接受有一定的困难，换句话说区块链应用向用户推广的难度偏高，想要获取用户的首次点击尝试变得相当困难。所以我们提出了`基于账户原型的交易费用代付协议`，来降低终端用户使用区块链应用的成本。

<!-- more -->

## Prototype - 原型

在最初开始的时候它并没有一个响亮的名字，在内部它被称作`原型`。一个超级简略的名字，但是这个名字确实反映出来设计它的初衷。这里所讨论的原型是针对账户的原型，所有的账户都会基于`原型`来构造并自动继承它的属性。这个`原型`其实就是 VeChain 对于 Ethereum 的账户模型的扩展。我们知道在 Ethereum 中账户两种：内部账户（合约）和外部账户（私钥控制的账户）， 有以下属性: 地址、余额、代码、存储。 我们在此基础上添加了一些概念（包括但不限于，这里仅讨论与代付协议相关的内容）。

+ `Master`： 地址对应的控制账户，一个抽象出来的类似于Owner的概念。外部账户的 Master 默认为它自己，合约账户则为部署交易的账户。
+ `User`：账户的用户，一般指合约的用户。
+ `Sponsor`：账户的赞助账户。
+ `CreditPlan`：合约为他的用户设定的额度，指合约会为他的用户承担的交易费的上限值。
+ `UserCredit`：用户的可用额度，合约的每个用户都会有一个当前的可用额度。
+ `RecoverRate`：恢复速率，当用户的可用额度低于设定的额度时，可以以一定的速率恢复，可以为0。

理解了上述概念，接下来理解 `MPP` 就很轻松了。

## 多方支付协议

对于代付协议来说，很重要的一个流程就是交易费抵扣的部分：

+ 检查交易的 `Clauses` 是否共享同一个接收账户(`CommonTo`)
+ 检查 `TxOrigin` 是否为用户 `CommonTo` 的用户
+ 检查用户额度
+ 检查赞助账户是否有足够的余额
+ 检查 `CommonTo` 的账户是否有足够的余额（优先从赞助账户扣除）

如果以上检查都通过，则此交易为被代付的交易，最终会体现为交易的 `GasPayer ≠ TxOrigin`， 这个时候 `GasPayer` 可能是 `CommonTo` 的赞助者或者 `CommonTo` 它本身

## 应用场景以及短板

`基于账户原型的交易费用代付协议`可以初步降低开发者引入用户的成本,让用户免费调用应用成为了可能。想象一下开发者通过几步对于`原型`的配置就可以让用户使用区块链应用，他可以实现{% label info@一次配置处处生效 %}。~~我认为~~这是区块链应用中杀手级的协议了:-)，~~有机会~~让更多的用户来了解来使用区块链应用。

好了，夸了这么多也该聊聊他的缺点了：

1. 由于交易手续费的扣除的层次是交易而非子句，所以可以被代付的交易的前提就是要有`CommonTo`
2. 在使用之前开发者需要进行配置，并且需要把每个用户都添加到合约的用户列表中

总结起来，就是

{% note warning %}
它不够灵活且无法覆盖到所有的应用场景
{% endnote %}