---
title: "Next.jsで今の場所（current url）を判定して表示を変えるときにつまづいた話"
emoji: "🗂"
type: "tech"
topics:
  - "nextjs"
published: true
published_at: "2023-05-05 05:49"
---

[Next.jsで今の場所（current url）を判定して表示を変える](https://zenn.dev/k_neko3/articles/893c2409f405b0)
こちらの記事を拝読し実際に実装した際に、私がつまづいた話です。

## やりたいこと

現在の場所を判定して、ナビゲーションの該当場所の表示を変えたい
![正しく動いているナビゲーションの画像](https://storage.googleapis.com/zenn-user-upload/68f586dba050-20230505.png)

## つまづいたこと

どのページに飛んでも、該当場所に加えて「ホーム」の表示が変わってしまう
![間違った動きをしているナビゲーションの画像](https://storage.googleapis.com/zenn-user-upload/d7d545afcc7c-20230505.png)

## 解決後コード全文

```jsx:Nav.jsx
import Link from "next/link";
import { useRouter } from "next/router";
import styles from "./Nav.module.css";

export default function Nav() {
  const router = useRouter();
  const navItems = [
    {
      href: "/",
      title: "ホーム",
    },
    {
      href: "/news",
      title: "お知らせ",
    },
    {
      href: "/about",
      title: "概要",
    },
    {
      href: "/join",
      title: "メンバー募集",
    },
    {
      href: "/contact",
      title: "お問い合わせ",
    },
    {
      href: "/privacy",
      title: "当サイトについて",
    },
  ];

  return (
    <nav>
      <ul>
        {navItems.map((item) => {
          return (
            <li key={item.href}>
              <Link
                href={item.href}
                className={
                  (
                    item.href === "/"
                      ? router.pathname === item.href
                      : router.pathname.startsWith(item.href)
                  )
                    ? styles.menuContentListItemLinkCurrent
                    : styles.menuContentListItemLink
                }
              >
                {item.title}
              </Link>
            </li>
          );
        })}
      </ul>
    </nav>
  );
}
```

## 原因

classを切り替える条件について、適切な条件を与えられていなかったことが原因です。
初めは先行記事に則り[startsWith()メソッド](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/String/startsWith)のみを使用していました。

```js
router.pathname.startsWith(item.href)
  ? styles.menuContentListItemLinkCurrent
  : styles.menuContentListItemLink
```

しかしこれは「/news」も「/」で始まるため、予期せぬ場所の表示が変わっていました。

|item.href|router.pathname|result|
|---------|---------------|------|
|/        |/news          |**true（！？）**|
|/news    |/news          |true  |
|/about   |/news          |false |
|/join    |/news          |false |
|/contact |/news          |false |
|/privacy |/news          |false |

## 解決策

item.hrefが「/」のときは、startsWithの代わりに===（厳密等価演算子）を使って判定するようにして解決です。

```diff js
- router.pathname.startsWith(item.href)
+ (
+   item.href === "/"
+     ? router.pathname === item.href
+     : router.pathname.startsWith(item.href)
+ )
    ? styles.menuContentListItemLinkCurrent
    : styles.menuContentListItemLink
```
