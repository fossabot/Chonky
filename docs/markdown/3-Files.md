The `FileBrowser` component accepts a property called `files`, which should be an array of file objects. It also the 
only required property, so the most basic Chonky usage could look like this:
 
```jsx
export const ExampleComponent = () => {
    const myFiles = [];
    return <FileBrowser files={myFiles}/>
}
```

### File object type

Each object in the `files` array should describe a file. To be precise, each object should satisfy the `FileData` 
type. The full definition of `FileData` type can be seen below (taken from
[TypeScript definitions](https://github.com/TimboKZ/Chonky/blob/master/src/typedef.ts)).

```typescript { "typeName" : "FileData" }
```

The main idea to take away is that each object must have an `id` and `name` defined. All of the other properties are 
optional, but they could significantly improve user experience if they are present.

If you don't specify the file extension (`ext` property), extension will be extracted automatically using
`path.extname(...)` function from the `path` module. If you dont want Chonky to automatically detect the extension, you 
can set `ext` to an empty string: `file.ext = ''`. If you do specify an extension, make sure it matches the actual 
extension of the file and starts with a dot, e.g. `file.ext = '.tar.gz'`.

#### Custom file properties

You are also free to add custom properties to objects, e.g. `file.encryption = 'RSA'`. `FileBrowser` will just 
ignore them, but they might be useful for your action handlers.

#### Loading placeholders

If you want to have a nice loading animation, you can pass several `null` values instead of a file object. For example,
if you know that a directory has 6 files in total, but only 2 of them were loaded, your `files` array could look like
this:

```js
const files = [myFile1, myFile2, null, null, null, null];
``` 

This configuration will render 2 files and 4 loading placeholders. Check the examples below to get a better idea.

### Immutability

To prevent useless re-renders and maximize performance, Chonky treats the list of files you pass in as an [immutable 
data structure](https://wecodetheweb.com/2016/02/12/immutable-javascript-using-es6-and-beyond/). If you're familiar 
with Redux, you're probably already relying on immutability in your code. If you haven't used Redux, it is still a 
good idea to read the [Immutable Data](https://redux.js.org/faq/immutable-data) section of Redux documentation to 
understand why it is useful.

For you, immutability mostly means that you cannot update file objects directly. Instead, you have to create a new 
copy of the object every time you want to make some changes.

Consider the following example: Your React component state holds an array called `files`, which contains 3 files. You
have a function that renames the second file to some predefined value, let's call it `renameSecond()`. You might be 
tempted to update your state in a way shown below.

```js
// BAD PRACTICE: Don't do this.
class BadComponent extends React.Component {
    //...
    renameSecond() {
        this.setState(prevState => {
            const files = prevState.files;
            // We change the object directly - this is bad!
            files[1].name = 'MyFile.md';
            return {files};
        })
    }
}
```

This is a bad practice because it changes the second file object directly. It also mutates the `files` array, instead
of creating a new copy. As a result, passing `this.state.files` to `FileBrowser` will not actually update the rendered 
file name. The correct way to update state in this case looks like this:

```js
// GOOD PRACTICE: follow this example
class GoodComponent extends React.Component {
    //...
    renameSecond() {
        this.setState(prevState => {
            // Create a copy of the array using ES6 spread operator
            const files = [...prevState.files];
            
            // Create a copy of the file object (using spread operator)
            // and overwrite relevant properties.
            files[1] = {
                ...files[1],
                name: 'MyFile.md',
            };
            
            // Update state
            return {files};
        })
    }
}
```

This way, the immutability properties are satisfied, and Chonky will correctly detect that the second object has 
changed and re-render file names as necessary.

### Example file list

We'll use the following `files` array:

```js
const files = [
    null,
    null,
    {
        id: 'zxc',
        name: 'Hidden file.mp4',
        isDir: false,
        isHidden: true,
        size: 890,
    },
    {
        id: 'bnm',
        name: 'Normal folder',
        isDir: true,
        childrenIds: ['random-id-1', 'random-id-2']
    },
    {
        id: 'vfr',
        name: 'Symlink folder',
        isDir: true,
        isSymlink: true,
    },
    {
        id: 'qwe',
        name: 'Not selectable.tar.gz',
        ext: '.tar.gz', // Custom extension
        selectable: false, // Disable selection
        size: 54300000000,
        modDate: new Date(),
    },
    {
        id: 'rty',
        name: 'Not openable.pem',
        openable: false, // Prevent opening
        size: 100000000,
    },
];
```

Note how each file is behaves differently based on its boolean flags:

```js { "componentPath" : "../components/Files.js" }
```
