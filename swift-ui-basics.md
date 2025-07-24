# swift ui basics

Some basics about SwiftUi:
- You will need a mac and xcode. duh. i followed this https://www.swift.org/getting-started/swiftui/ docs and it was lovely.
- NIL in swift UI means the absence of a value.
- Swift comes with a bunch of default fundation elements for the UI, like `Circle` and `Text`.
- We have things called <code>View modifiers</code> that we can apply to the UI elements to change their style and the way they look.
For example:

```
var body: some View {
    Circle()
        .fill(.blue)
        .padding()
        .overlay(
            Image(systemName: "figure.archery")
                .font(.system(size: 144))
                .foregroundColor(.white)
        )

    Text("Archery!")
        .font(.title)
}
```

- oh, we can nest stacks! very cool. using `VStack` for example.

```
VStack {
    Text("Why not try…")
        .font(.largeTitle.bold())

    VStack {
        Circle()
            .fill(.blue)
            .padding()
            .overlay(
                Image(systemName: "figure.archery")
                    .font(.system(size: 144))
                    .foregroundColor(.white)
            )

        Text("Archery!")
            .font(.title)
    }
}
```