# kotori

Just some random thoughts...

## Main config

```js
{
id: 1,
entity: 'kotori/core',
seed: 'blabla'
masterKeys: [
 { type: 'bearer', key: '823823058' },
 { type: 'bearer', key: '823823493' },
]
}
```

## Projects

```js
{
id: 2,
entity: 'kotori/project',
coreId: 1,
name: 'My blog',
keys: [
 { type: 'bearer', key: '823823058' },
 { type: 'bearer', key: '823823493', isReadOnly: true },
 { type: 'bearer', key: '823823494', isReadOnly: true, drafts: true, future: true }
]
}
```

## Document types

```js
{
entity: 'kotori/document-type',
projectId: 2,
types: [
 { type: '/content/review', indexMappings: [
  { from: 'title', to: 'title' },
  { from: 'author', to: 'text2' }
 ]},
 { type: '/content/news', indexMappings: [] }
]
}

```

## Document 

```js
{
id: [guid]
slug: 'review/fringe',
entity: 'kotori/document',
projectId: 2,
documentType: '/content/review',
isDraft: false,
isPublished: false,
date: {
      year: 2017,
      month: 6,
      day: 22,
      hour: 0,
      minute: 0,
      second: 0,
      dateTime: "2017-06-22T00:00:00",
      epoch: 1498089600
    },
attributes: [
   genre: 'sci-fi'
],
content: '# hello',
hash: [sha256]
}
}


```
