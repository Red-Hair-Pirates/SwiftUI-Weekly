# Working with files and folders in Swift

[Working with files and folders in Swift](https://swiftbysundell.com/articles/working-with-files-and-folders-in-swift/)    作者：[[Swift with Majid](https://swiftwithmajid.com/)]

翻译：阳阳

Reading and writing files and folders is one of those tasks that almost every app needs to perform at some point. While many apps these days, especially on iOS, might not give users transparent access to open, save, and update documents as they please — whenever we’re dealing with some form of long-term data persistence, or a bundled resource, we always have to interact with the file system one way or another.

So this week, let’s take a closer look at various ways to use the many file system-related APIs that Swift offers — both on Apple’s own platforms, and on platforms like Linux — and a few things that can be good to keep in mind when working with those APIs.

URLs, locations, and data
Fundamentally, there are two Foundation types that are especially important when reading and writing files in Swift — URL and Data. Just like when performing network calls, URLs are used to point to various locations on disk, which we can then either read binary data from, or write new data into.

For example, here we’re retrieving a file path passed as an argument to a Swift command line tool, which we then turn into a file system URL in order to load that file’s data:

// This lets us easily access any command line argument passed
// into our program as "-path":
guard let path = UserDefaults.standard.string(forKey: "path") else {
    throw Error.noPathGiven
}

let url = URL(fileURLWithPath: path)

do {
    let data = try Data(contentsOf: url)
    ...
} catch {
    throw Error.failedToLoadData
}
To learn more about the above way of using UserDefaults, and using command line arguments in general, check out “Launch arguments in Swift”.

One thing that’s typically good to keep in mind when working with string-based paths is that certain characters are expected to be interpreted in specific ways, such as the tilde character (~), which is commonly used to refer to the current user’s home directory.

While that’s not something that we typically have to handle manually when dealing with command line tool input (as terminal shells tend to expand such symbols automatically), within other contexts we can enlist the help of the String type’s Objective-C “cousin”, NSString, to help us expand any tilde character found within a given string into the user’s full home directory path:

var path = resolvePath()
path = (path as NSString).expandingTildeInPath
Worth noting is that NSString is also available on Linux, through the open source, Swift-based version of Foundation.

Bundles and modules
On Apple’s platforms, apps are distributed as bundles, which means that in order to access internal files that we’ve included (or bundled) within our own app, we’ll first need to resolve their actual URLs by searching for them within our app’s main bundle.

That main bundle can be accessed using Bundle.main, which lets us retrieve any resource file that was included within our main app target, such as a bundled JSON file, like this:

struct ContentLoader {
    enum Error: Swift.Error {
        case fileNotFound(name: String)
        case fileDecodingFailed(name: String, Swift.Error)
    }

    func loadBundledContent(fromFileNamed name: String) throws -> Content {
        guard let url = Bundle.main.url(
            forResource: name,
            withExtension: "json"
        ) else {
            throw Error.fileNotFound(name: name)
        }

        do {
            let data = try Data(contentsOf: url)
            let decoder = JSONDecoder()
            return try decoder.decode(Content.self, from: data)
        } catch {
            throw Error.fileDecodingFailed(name: name, error)
        }
    }
    
    ...
}
While it might at first seem like Bundle.main is the only bundle that we’ll ever need to work with, that’s typically not the case. For example, let’s say that we now want to write a unit test that verifies the above ContentLoader by having it load specific file that was bundled within our test bundle:

class ContentLoaderTests: XCTestCase {
    func testLoadingContentFromBundledFile() throws {
        let loader = ContentLoader()
        let content = try loader.loadBundledContent(fromFileNamed: "testContent")
        XCTAssertEqual(content.title, "This is a test")
    }
    
    ...
}
When running the above test, we’ll end up getting an error, which might initially seem a bit strange (assuming that we’ve bundled a file called testContent.json within our test target). The problem is that our unit testing suite has its own bundle, that’s separate from Bundle.main, and since our ContentLoader currently always uses the main bundle, our test file won’t be found.

So, in order to be able to perform the above test, we first need to add a bit of parameter-based dependency injection to enable ContentLoader to load files from any Bundle (while still keeping main as the default):

struct ContentLoader {
    ...

    func loadBundledContent(fromFileNamed name: String,
                            in bundle: Bundle = .main) throws -> Content {
        guard let url = bundle.url(
            forResource: name,
            withExtension: "json"
        ) else {
            throw Error.fileNotFound(name: name)
        }

        ...
    }
    
    ...
}
With the above in place, we can now resolve the correct bundle within our unit tests — by asking the system for the bundle that contains our current test class — which we’ll then inject when calling our loadBundledContent method:

class ContentLoaderTests: XCTestCase {
    func testLoadingContentFromBundledFile() throws {
        let loader = ContentLoader()
        let bundle = Bundle(for: Self.self)

        let content = try loader.loadBundledContent(
            fromFileNamed: "testContent",
            in: bundle
        )

        XCTAssertEqual(content.title, "This is a test")
    }
    
    ...
}
Along those same lines, when using the Swift Package Manager’s new (as of Swift 5.3) capability that lets us embed bundled resources within a Swift package, we also can’t assume that Bundle.main will contain all of our app’s resources — since any file bundled within a Swift package will be accessible through the new module property, which refers to the current module’s bundle, rather than the one for the app itself.

So, in general, whenever we’re designing an API that uses Bundle to load local resources, it’s typically a good idea to enable any Bundle instance to be injected, rather than hard-coding our logic to always use the main one.

Storing files within system-defined folders
So far, we’ve been exploring various ways to read files, either from any file system location through a command line tool (running on either macOS or Linux), or from a file bundled within an application. But now, let’s take a look at how we can write files as well — in a way that’s both predictable, and compatible with the tighter sandboxing rules found on platforms like iOS.

Actually writing binary data to disk is as easy as calling the write(to:) method on any Data value, but the question is how to resolve what URL to write to — especially if we want to write a file to a system-defined folder, such as Library or Documents.

The answer is to use Foundation’s FileManager API, which lets us resolve URLs for system folders in a cross-platform manner. For example, here’s how we could encode and write any Encodable value to file within the current user’s Documents folder:

struct FileIOController {
    func write<T: Encodable>(
        _ value: T,
        toDocumentNamed documentName: String,
        encodedUsing encoder: JSONEncoder = .init()
    ) throws {
        let folderURL = try FileManager.default.url(
            for: .documentDirectory,
            in: .userDomainMask,
            appropriateFor: nil,
            create: false
        )

        let fileURL = folderURL.appendingPathComponent(documentName)
        let data = try encoder.encode(value)
        try data.write(to: fileURL)
    }
    
    ...
}
On macOS, the above folderURL will point to ~/Documents, just as we’d expect, but on iOS it’ll instead point to our app’s own version of that folder that’s located within the app’s sandbox.

Similarly, we can also use the above FileManager API to resolve other kinds of system folders as well — for example the folder that the system deems the most appropriate to use for disk-based caching:

let cacheFolderURL = try FileManager.default.url(
    for: .cachesDirectory,
    in: .userDomainMask,
    appropriateFor: nil,
    create: false
)
If all that we’re looking for is a URL for a temporary folder, however, we can use the much simpler NSTemporaryDirectory function — which returns a URL for a system folder can be used to store data that we only wish to persist for a short period of time:

let temporaryFolderURL = URL(fileURLWithPath: NSTemporaryDirectory())
The same URL can also be retrieved using FileManager.default.temporaryDirectory.

The benefit of using the above APIs, rather than hard-coding specific folder paths within our code, is that we’re letting the system decide what folders that are the most appropriate for the task at hand, which typically goes a long way toward making code dealing with the file system more portable and much more future-proof.

Managing custom folders
Although storing files directly within folders that are managed by the system does have its use cases, chances are that we’ll instead want to encapsulate our files within a folder of our own — specially when writing files to shared system folders (such as Documents or Library) on macOS, which could cause conflicts with other apps or user data if we’re not careful.

This is another area in which FileManager is really useful, as it provides a number of APIs that let us create, modify and delete custom folders. For example, here’s how we could modify our FileIOController from before to now store its files within a nested MyAppFiles folder, rather than within the Documents folder directly:

struct FileIOController {
    var manager = FileManager.default

    func write<T: Encodable>(
        _ object: T,
        toDocumentNamed documentName: String,
        encodedUsing encoder: JSONEncoder = .init()
    ) throws {
        let rootFolderURL = try manager.url(
            for: .documentDirectory,
            in: .userDomainMask,
            appropriateFor: nil,
            create: false
        )

        let nestedFolderURL = rootFolderURL.appendingPathComponent("MyAppFiles")

        try manager.createDirectory(
            at: nestedFolderURL,
            withIntermediateDirectories: false,
            attributes: nil
        )

        let fileURL = nestedFolderURL.appendingPathComponent(documentName)
        let data = try encoder.encode(object)
        try data.write(to: fileURL)
    }
    
    ...
}
The above code does have a quite major problem, though, and that’s that we’re currently attempting to create our nested folder every time that our write method is called — which will cause an error to be thrown if that folder already exists.

While we could simply prefix our call to createDirectory with try?, rather than try, to fix that problem — doing so would also silence any legitimate errors that could be thrown when we actually want to create that folder, which wouldn’t be ideal. So let’s instead use another FileManager API, fileExists, which can also be used to check if a folder exists at a given path:

if !manager.fileExists(atPath: nestedFolderURL.relativePath) {
    try manager.createDirectory(
        at: nestedFolderURL,
        withIntermediateDirectories: false,
        attributes: nil
    )
}
An optional isDirectory parameter can also be passed to the fileExists method if we’d also also like to check if the item at the given path is indeed a folder, but doing so feels a bit redundant in the above case.

Note how we’re using the relativePath property to convert our above nestedFolderURL to a string-based path, rather than using absoluteString, which is typically used when working with URLs pointing to a location on the internet. That’s because absoluteString would yield a URL string prefixed with the file:// scheme, which is not what we want when passing a file URL to an API that accepts a file path.

Also worth noting is that the above approach is really only safe within single-threaded contexts, or when our program is in complete control over the directories that it creates, since otherwise there’s a risk that the folder in question could end up being created in between our fileExists check and our call to createDirectory. One way to handle such situations would be to always try to create the directory, and then ignore any resulting error only if that error matches the one thrown when a folder already existed — like this:

do {
    try manager.createDirectory(
        at: nestedFolderURL,
        withIntermediateDirectories: false,
        attributes: nil
    )
} catch CocoaError.fileWriteFileExists {
    // Folder already existed
} catch {
    throw error
}

Conclusion
Swift, and more specifically Foundation, ships with a quite comprehensive suite of file system APIs that enable us to perform a large number of operations in ways that work across all of Apple’s platforms — and many of them are also fully Linux-compatible as well. While this article didn’t aim to cover every single API (that’s what Apple’s official documentation is for, after all), I hope that it has provided a somewhat concise overview of the various key APIs that are involved when it comes to working with files and folders in Swift.

For practical examples of some of the above APIs, and many more, feel free to also check out my Files library, which acts as an object-oriented wrapper around system APIs like FileManager. And, if you have questions, comments, or feedback, then you’re always welcome to contact me.

Thanks for reading! 🚀