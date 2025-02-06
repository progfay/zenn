---
title: '"Cannot find matching keyid" in Corepack'
emoji: "🗝️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Corepack"]
published: true
---

本記事は 2025/01/27 頃に発生した Corepack 経由で Package Manager を install した際に `Cannot find matching keyid` というエラーが発生した事象について解説するものです。

https://github.com/nodejs/corepack/issues/612

## 解決方法

問題の解決は主に以下の 2 つになります。

- Corepack を [`v0.31.0`](https://github.com/nodejs/corepack/releases/tag/v0.31.0) に上げる
- Corepack の利用をやめる

## 原因

npm Registry は先日 [Public signing keys](https://registry.npmjs.org/-/npm/v1/keys) の rotate を行いました。 (理由などの詳細は未調査)

一方 Corepack は Package Manager を install する際に [package の署名を検証](https://docs.npmjs.com/verifying-registry-signatures) します。

しかし rotate されたことにより新しく追加された key 情報を Corepack 側が知らなかったため、新しい key を使って署名された package の検証に失敗しました。

## 詳細

Corepack は package manager の install 時に `verifySignature` 関数を実行します。

https://github.com/nodejs/corepack/blob/v0.30.0/sources/npmRegistryUtils.ts#L35

この関数の処理は以下のとおりです。

1. package の情報から signature を取得する
2. Corepack 自身が持つ key 情報と合致する key と signature を探す
3. 署名の検証を行う

## 1. package の情報から signature を取得する

Corepack は `https://registry.npmjs.org/${PACKAGE_NAME}/${VERSION}` の `dist` field から情報を取得します。
ここでは `pnpm@10.1.0` を例に挙げてみましょう (一部抜粋):

```json
{
  "dist": {
    "shasum": "ab7948c89104fdd3fc88b5b391fa4b73fd800631",
    "tarball": "https://registry.npmjs.org/pnpm/-/pnpm-10.1.0.tgz",
    "fileCount": 1206,
    "integrity": "sha512-yJhHsGZ92rUDlru9AIoqQ887WB79Wc9dmqiSPqH7S4EGwEHVQNCKywlQN1lNc+vFHh7InuQMiLMLimbA+uCsGw==",
    "signatures": [
      {
        "sig": "MEUCIQDlkgmNyZjT7KUY8AO6jH7Gs3fyiXG8nbTnuLbd8fOS2AIgXyJ6SaYhumMFzUYQAZPJGhsnlaD5N0X2MZsbG+eS/Xo=",
        "keyid": "SHA256:DhQ8wR5APBvFHLF/+Tc+AYvPOdTpcIDqOhxsBHRwC7U"
      }
    ],
    "unpackedSize": 18870570
  }
}
```

## 2. Corepack 自身が持つ key 情報と合致する key と signature を探す

Corepack は npm の Public signing keys を local に持っています。

[`v0.30.0`](https://github.com/nodejs/corepack/blob/v0.30.0/config.json) 時点では以下のような内容になっていました (一部抜粋):

```json
{
  "keys": {
    "npm": [
      {
        "expires": null,
        "keyid": "SHA256:jl3bwswu80PjjokCgh0o2w5c2U4LhQAE57gj9cz1kzA",
        "keytype": "ecdsa-sha2-nistp256",
        "scheme": "ecdsa-sha2-nistp256",
        "key": "MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE1Olb3zMAFFxXKHiIkQO5cJ3Yhl5i6UPp+IhuteBJbuHcA5UogKo0EWtlWwW6KSaKoTNEYL7JlCQiVnkhBktUgg=="
      }
    ]
  }
}
```

`corepack@0.30.0` と `pnpm@10.1.0` が持つ Key ID は以下のようになっていました:

| Package           | Key ID                                               |
| ----------------- | ---------------------------------------------------- |
| `corepack@0.30.0` | `SHA256:jl3bwswu80PjjokCgh0o2w5c2U4LhQAE57gj9cz1kzA` |
| `pnpm@10.1.0`     | `SHA256:DhQ8wR5APBvFHLF/+Tc+AYvPOdTpcIDqOhxsBHRwC7U` |

2025/02/06 時点で https://registry.npmjs.org/-/npm/v1/keys は以下のような内容になっています:

```json
{
  "keys": [
    {
      "expires": "2025-01-29T00:00:00.000Z",
      "keyid": "SHA256:jl3bwswu80PjjokCgh0o2w5c2U4LhQAE57gj9cz1kzA",
      "keytype": "ecdsa-sha2-nistp256",
      "scheme": "ecdsa-sha2-nistp256",
      "key": "MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE1Olb3zMAFFxXKHiIkQO5cJ3Yhl5i6UPp+IhuteBJbuHcA5UogKo0EWtlWwW6KSaKoTNEYL7JlCQiVnkhBktUgg=="
    },
    {
      "expires": null,
      "keyid": "SHA256:DhQ8wR5APBvFHLF/+Tc+AYvPOdTpcIDqOhxsBHRwC7U",
      "keytype": "ecdsa-sha2-nistp256",
      "scheme": "ecdsa-sha2-nistp256",
      "key": "MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEY6Ya7W++7aUPzvMTrezH6Ycx3c+HOKYCcNGybJZSCJq/fd7Qa8uuAKtdIkUQtQiEKERhAmE5lMMJhP8OkDOa2g=="
    }
  ]
}
```

これを見るに Corepack は古い key ID のみを保持しており、 `pnpm@10.1.0` は新しい key で署名されていることがわかります。

| Package           | Key ID                                               |        |
| ----------------- | ---------------------------------------------------- | ------ |
| `corepack@0.30.0` | `SHA256:jl3bwswu80PjjokCgh0o2w5c2U4LhQAE57gj9cz1kzA` | OLD 🗝️ |
| `pnpm@10.1.0`     | `SHA256:DhQ8wR5APBvFHLF/+Tc+AYvPOdTpcIDqOhxsBHRwC7U` | NEW 🔑 |

### 3. 署名の検証を行う

`verifySignature` では Corepack と install する package 側で同一の key を持っていない場合に `new Error("Cannot find matching keyid")` が throw されます。

https://github.com/nodejs/corepack/blob/v0.30.0/sources/npmRegistryUtils.ts#L45-L48

## 修正

今回の問題は Corepack が持つ npm の Public signed key が古い状態だったことが原因であるため、これを更新することで解決しました。

https://github.com/nodejs/corepack/pull/614

この修正は `corepack@0.31.0` としてリリースされました。
