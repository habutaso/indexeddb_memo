# indexedDBでできる事，ラッパーライブラリ
## 目次
1. indexedDBでできること
    + データベースの新規作成，接続
    + データベースの削除
    + オブジェクトストア(テーブル)の追加
    + データ
      + データ挿入
      + データ取得
      + データ削除
      + データ検索
          + index
          + cursor
2. ラッパーライブラリ
    + dexie.js
    + idb


## indexedDBでできること

indexedDBは非同期で実行される．成功したら`onsuccess`が，失敗したら`onerror`が実行される．

### データベース新規作成, 接続

```typescript
const dbName = 'database'

const request = indexedDB.open(dbName)

request.onupgradeneeded = (event) => {
  // indexedDBのバージョン更新や新規作成時に実行する
}

request.onsuccess = (event) => {
  // indexedDBの接続に成功したら実行する
}

request.onerror = (event) => {
  // indexedDBの接続に失敗したら実行する
}
```

### データベースの削除

```typescript
const request = indexedDB.deleteDatabase(dbName)

request.onsuccess = (event) => {
  // dbNameが存在しないデータベースであってもこちらが実行される
}

request.onerror = (event) => {
  // 削除エラー時
}
```

### オブジェクトストアの作成

```typescript
const storeName = 'storename'
// スキーマバージョニングという概念がある
// アプリリリース後などで，オブジェクトストアを追加/削除したいときに使用する．
// 前回のバージョン整数+1をすることで，オブジェクトストアに変更を与えることができる．
const dbVersion = 1

const request = indexedDB(dbName, dbVersion)

request.onupgradeneeded = (event) => {
  const db = event.target.result
  db.createObjectStore(storeName, { keyPath: 'id' })
}
```

### データ
基本的な更新の流れは
1. トランザクションを作成する
2. オブジェクトストアを選択する
3. データCRUDアクションを行う


### データ挿入

```typescript
const sampleData = { id: 1, name: 'sampleName', favorite: 'grape' }

const request = indexedDB.open(dbName)

request.onsuccess = (event) => {
  const db = event.target.result
  // データ変更時にはトランザクションを作成する．
  // 複数のオブジェクトストアをロックすることができる
  const transaction = db.transaction([storeName1, storeName2, ...], 'readwrite') // 'readonly'もある
  // 更新をかけたいオブジェクトストアを読み込む
  const store = transaction.objectStore(storeName)

  // データ追加だけならadd
  const addRequest = store.add(sampleData)
  addRequest.onsuccess = (event) => {
    // 成功時
  }
  addRequext.onerror = (event) => {
    // 失敗時
  }

  // データ追加/更新ならput
  const putRequest = store.put(sampleData)
  putRequest.onsuccess = (event) => {
    // 成功時
  }
  putRequext.onerror = (event) => {
    // 失敗時
  }

}
```

### データ取得

```typescript
const key = 1

const request = indexedDB.open(dbName)

request.onsuccess = (event) => {
  const db = event.target.result
  // データ取得時にはトランザクションを作成する．
  // 複数のオブジェクトストアをロックすることができる
  const transaction = db.transaction([storeName1, storeName2, ...], 'readonly')
  // 取得したいオブジェクトストアを読み込む
  const store = transaction.objectStore(storeName)

  const getRequest = store.get(key)

  getRequest.onsuccess = (event) => {
    console.log(event.target.result) // { id: 1, name: 'sampleName', favorite: 'grape' }
  }
}
```