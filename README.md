# PoC-MutationObserver-on-WebComponent

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width" />
  <title>&lrm;</title>
  <link rel="icon" href="data:image/x-icon;base64,AA">
</head>
<body>

  <button id="add-item">Add item</button>

  <div id="items"></div>

  <script>
    document.getElementById("add-item").onclick = () =>
      document.getElementById("items").appendChild(document.createElement("my-item"))
  </script>

  <template id="my-item">
    <div>item</div>
  </template>

  <script>
    class MyItem extends HTMLElement {
      static tagName = "my-item"
      constructor() {
        super()
        this.attachShadow({ mode: "open" }).appendChild(
          document.querySelector("template#my-item").content.cloneNode(true)
        )
      }
    }

    const myWebComponents = new Map()
    myWebComponents.set(MyItem.tagName, MyItem)

    const observer = new MutationObserver((mutationList) => {
      for (const mutation of mutationList) if (mutation.type === "childList")
        mutation.addedNodes.forEach(addedNode => {
          const tagName = addedNode.tagName?.toLowerCase()
          if (customElements.get(tagName)) return
          const Component = myWebComponents.get(tagName)
          if (Component) {
            console.info("Define custom element", tagName)
            customElements.define(tagName, Component)
          }
        })
    })

    observer.observe(document.body, { attributes: false, childList: true, subtree: true });
  </script>

</body>
</html>
```
