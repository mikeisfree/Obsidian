---
title: '"Mermaid Syntax & Examples"'
draft: false
tags:
  - tutorial
  - mermaid
  - cheatsheet
vault: mike
---
 


```mermaid
flowchart TD
  A[The Excalidraw Plugin is Community Supported] --> B{Will YOU support it?}
  B -- 👍 Yes --> C[Long-term stability + new features]
  B -- No 👎 --> D[Plugin eventually stops working 😢]
  C --> E[Support at ❤️ https://ko-fi.com/zsolt]
  E --> F[📢 Encourage others to support]
  D --> G[🪦 R.I.P. Tryout Plugin]
  ```



---



```mermaid
flowchart LR

id1[This is the text in the box]
```

---
  

### Diagrams

  

---
 



  

```mermaid

graph TD;

    A-->B;

    A-->C;

```

  

### Sequence Diagrams

  

```mermaid

sequenceDiagram

    participant Alice

    participant Bob

    Alice->>John: Hello John, how are you?

    John-->>Alice: Great!

    John->>Bob: How about you?

    Bob-->>John: Jolly good!

```


---

```mermaid
flowchart LR 
id["This ❤ Unicode"]
```




