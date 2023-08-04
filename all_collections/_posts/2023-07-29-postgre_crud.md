---
layout: post
title: 🐘 Postgre로 CRUD를 해보자
date: 2023-07-29
categories: ["SamBeak", "Postgre", "AWS", "VPC", "Node", "DB", "DATABASE"]
---

> # PostgreSQL 데이터베이스 만들기

<br>
첫 단계는 데이터베이스를 만들어보는 것이다. <br>
어렵지는 않지만, 그렇다고 MySQL과는 다르니 <br>
주의하고 접근하는 것이 좋겠다. <br><br>

PostgreSQL의 데이터베이스를 만드는 것은 <br>
with가 들어간다는 것이 특이점이다. <br>

```SQL
CREATE DATABASE db_name WITH ENCODING='UTF-8';
```

> # PostgreSQL 테이블 만들기

<br>
기존에 만들었던 MySQL과 동일하게 작성한다. <br>
그러나, 다른 점은 PostgreSQL은 UUID가 지원되기 때문에 <br>
별도로 BIN_TO_UUID같은 과정을 해주지 않아도 된다. <br>
uuid는 구분해주기 위해 더블쿼트를 써주는 것이 좋다. <br><br>

```SQL
CREATE TABLE notes(
        "uuid" UUID DEFAULT gen_random_uuid(),
        title VARCHAR NOT NULL,
        contents TEXT NOT NULL,
        created TIMESTAMP DEFAULT NOW(),
        PRIMARY KEY ("uuid")
        );
```

> # PostgreSQL 데이터 INSERT

<br>
MySQL과 동일하게 작성하면 된다. <br>

```SQL
INSERT INTO notes (title, contents) VALUES ('title1', 'content1');
```

> # 본격 CRUD 만들기

<br>
이제부터 본격이라는 말을 썼는데 과정은 MySQL 때와 비슷하다. <br>
겁 먹지 말고 차근차근 가보자 <br><br>

본격이라 쓴 이유는 함수를 만들어 사용할 것이기 때문이다. <br>
getNotes, addNote, deleteNote 등 말이다. <br><br>

# getNotes

```JavaScript
async function getNotes(){
    const client = await pool.connect();
    const res = await client.query(
        `SELECT * FROM notes;`
    );
    return res.rows;
    client.release();
};

await getNotes();
```

# getNote

```JavaScript
async function getNote(id){
    const client = await pool.connect();
    const res5 = await client.query(
        `SELECT * FROM notes WHERE uuid = '${id}';`
    );
    return res5.rows;
    client.release();
};

await getNote();
```

# addNote

```JavaScript
async function addNotes(title, contents){
    const client = await pool.connect();
    const res6 = await client.query(
        `INSERT INTO notes (title, contents) VALUES ('${title}', '${contents}');`
    );
    return res6.rows;
    client.release();
};

await addNotes();
```

# updateNote

```JavaScript
async function updateNotes(id, title, contents){
    const client = await pool.connect();
    const res7 = await client.query(
        `UPDATE notes SET title = '${title}' AND contents = '${contents}' WHERE uuid = '${id}';`
    );
    return res7.rows;
    client.release();
};

await updateNotes();
```

# deleteNote

```JavaScript
async function deleteNotes(id){
    const client = await pool.connect();
    const res8 = await client.query(
        `DELETE FROM notes WHERE uuid = '${id}';`
    );
    return res8.rows;
    client.release();
};

async deleteNotes();
```
