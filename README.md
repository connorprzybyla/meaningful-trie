In today's coding tutorial, we will be implementing a [trie](https://en.wikipedia.org/wiki/Trie#:~:text=In%20computer%20science%2C%20a%20trie,key%2C%20but%20by%20individual%20characters.) data structure (also commonly known as a prefix tree) from scratch in Swift and understanding the advantages and disadvantages of this data structure. Let's first understand some basics.

Note: This tutorial assumes that you already have some basic level of understanding of commonly used data structures and asymptotic analysis. 

## What is a trie?

A trie can be thought of as a search tree in order to locate a specific pattern, or key from within a set.  It was invented in 1960 by three pioneers of computer science and combinatorics: Edward Fredkin, Axel Thue, and René de la Briandais.  

Let's take a look at a simple visual of what a trie might look like for the following set of strings. 

        Example 1:  ["food", "food close", "Food nearby", "pizza", "best Pizza"]
        
What can we observe/assume from the set above? Some initial thoughts might be:

  - There is no specific ordering of each element in the set.
  - The strings can vary by length.
  - Each element is unique.
  - Some characters are uppercased, others lowercased.
  
  
Now, try to imagine a data structure that would be best suited for locating all elements within this set that have a common prefix. 

               
           (root node)   
          /     |    \
        (f)    (p)   (b)      
         |      |     |
        (o)    (i)   (e) 
         |      |     |
        (o)    (z)   (s)
         |      |     |
        (d)    (z)   (t)
         |      |     |
        ( )    (a)   ( )
        / \           |
      (c) (n)        (p)
       |   |          |
      (l) (e)        (i)
       |   |          |
      (o) (a)        (z)
       |   |          |
      (s) (r)        (z)
       |   |          |
      (e) (b)        (a)
           |          
          (y)   


Consider the prefix: `foo`. Valid elements in the example set above with the same prefix (case insensitive) are `food`, `food close`, `Food nearby`.
    
## The problem we want to solve: 
We have a client that wants us to build a cutting-edge search engine so that users can find exactly what they are looking for within our application as quickly as possible. As a user types into our search bar, a prefix is formed that we want to search against our set of stored elements.

Naive solution: Why don't we just iterate through the set and compare the prefix with every element's prefix?

```swift

static func findMatch(_ prefix: String) -> [String] {

  let storedWord = ["food", "food close", "Food nearby", "pizza", "best Pizza"]
  
  /* Character arrays make life alot easier in Swift, 
     let's convert our input to one first. "foo" -> ["f", "o", "o"]
  */   
  let prefixChars = Array(prefix)
  
  // Result array to append all matches to
  var matches = [String]()
  
  for word in storedWord {
    let wordChars = Array(word)
    var hasSamePrefix = true
    for (index, char) in prefixChars.enumerated() {
      if char != wordChars[index] {
        hasSamePrefix = false
        break
      }
    }
    if hasSamePrefix == true {
      matches.append(word)
    }
  }
  
  return matches
}

```

Worst case time complexity of the above implementation: `O(storedWords * prefix)`
We must iterate through all stored words and at every iteration, we need to iterate through the characters of the stored word up to the length of the prefix.
     
Now, let's see how a trie works under the hood.
                       
```swift

import Foundation

// MARK: - TrieNode

final class TrieNode {
    var children: [Character: TrieNode]
    var isEnd: Bool
    
    init(
        children: [Character: TrieNode],
        isEnd: Bool
    ) {
        self.children = children
        self.isEnd = isEnd
    }
}

// MARK: - TrieProtocol

protocol TrieProtocol {
    var  root: TrieNode { get set }
    func insert(_ word: String)
    func search(_ word: String) -> Bool
    func startsWith(_ prefix: String) -> Bool
}

// MARK: - Trie

final class Trie: TrieProtocol {

    var root: TrieNode

    init() {
        root = TrieNode(
            children: [:],
            isEnd: false
        )
    }
    
    func insert(_ word: String) {
        var currentNode = root

        for char in word.lowercased() {
            if currentNode.children[char] == nil {
                currentNode.children[char] = TrieNode(
                    children: [:],
                    isEnd: false
                )
            }
            
            guard let childNode = currentNode.children[char] else {
                fatalError()
            }
            currentNode = childNode
        }
        
        currentNode.isEnd = true
    }
    
    func search(_ word: String) -> Bool {
        var currentNode = root
    
        for char in word.lowercased() {
            guard let childNode = currentNode.children[char] else {
                return false
            }
            
            currentNode = childNode
        }
        
        return currentNode.isEnd
    }
    
    func startsWith(_ prefix: String) -> Bool {
        var currentNode = root
        
        for char in prefix.lowercased() {
            guard let childNode = currentNode.children[char] else {
                return false
            }
            
            currentNode = childNode
        }
        
        return true
    }
}

```

How to write XCTest's for Trie:

```swift

import XCTest
@testable import DataStructuresAreFun

final class TrieTests: XCTestCase {

    private var sut: Trie!

    override func setUp() {
        super.setUp()
        sut = Trie()
    }
    
    override func tearDown() {
        sut = nil
        super.tearDown()
    }
    
    // MARK: - Search

    func testSearchReturnsTrue() {
        sut.insert("food")
        XCTAssertTrue(sut.search("food"))
    }
    
    func testSearchReturnsFalse() {
        sut.insert("pizza")
        XCTAssertFalse(sut.search("food"))
    }
    
    func testSearchReturnsFalseWithStringIsEmpty() {
        sut.insert("pizza")
        XCTAssertFalse(sut.search(""))
    }
    
    func testSearchReturnsFalseWhenStringAndTrieEmpty() {
        XCTAssertFalse(sut.search(""))
    }
    
    func testSearchReturnsFalseWhenEmptyTrie() {
        XCTAssertFalse(sut.search("food"))
    }
    
    // MARK: - StartsWith
    
    func testStartsWithReturnsTrue() {
        sut.insert("food")
        XCTAssertTrue(sut.startsWith("foo"))
    }
    
    func testStartsWithReturnsFalse() {
        sut.insert("pizza")
        XCTAssertFalse(sut.startsWith("foo"))
    }
    
    func testStartsWithReturnsFalseWhenStringIsEmpty() {
        sut.insert("pizza")
        XCTAssertFalse(sut.startsWith(""))
    }
    
    func testStartsWithReturnsFalseWhenStringAndTrieEmpty() {
        XCTAssertFalse(sut.startsWith(""))
    }
    
    func testStartsWithReturnsFalseWhenEmpty() {
        XCTAssertFalse(sut.startsWith("foo"))
    }
}

```
                       
Lot's more to come, stay tuned :)
- Asymptotic analysis of a trie
- Complex variations of tries 



  
