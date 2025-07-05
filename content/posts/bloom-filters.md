+++
date = '2025-07-05T05:30:00+05:30'
draft = false
title = "Bloom Filters: Fast, Probable and Powerful"
summary = "Large scale systems like databases and blockchains often need to check whether a key exists without doing a full scan of the dataset. Bloom filters provide a fast, memory-efficient way to rule out non-existent keys with zero false negatives and a small chance of false positives."
toc = true
readTime = true
autonumber = true
+++
## Problem
In large-scale systems, checking whether an item exist or not in a dataset can be expensive. Imagine querying for a non existent key among billions in a distributed database, or checking whether a blockchain node has already seen a transaction.

A naive approach is to scan the entire dataset — which is far too slow. Indexes can help to speed up query, but they often consume a significant amount of space, especially at scale.

**The challenge**: Is there a fast, space-efficient way to determine that an item is definitely not present?

This is where Bloom filters shine.

## How does bloom filter help
A bloom filter is a probabilistic data structure that efficiently answers whether a key exist in a dataset -- sacrificing precision for speed by allowing false positives, but **no false negatives**

- If it says a key is not in the dataset, it’s guaranteed to be absent (0% false negatives).

- If it says a key is in the dataset, it might be — there’s a small chance it’s a false positive.

## Here's how it works:
### EL5
Think of it as a small checklist. Instead of storing actual items, it stores fingerprints of items in a compact bit array. When you want to know whether something exists, it looks for those fingerprints.

### Step by Step
-  You initialize a bit array of size m with 0's
-  You define k independent hash functions
-  For every insert of key, pass it through k hash functions individually and mod it by m
-  The output of each hash function mod m will be an index. Set 1 at bitarray[index]
-  To check if a key exists, you hash it again and check the value of bitarray[index]. If any is 0, then the key doesnt exist. If all bits are 1, the key might exist
-  The more items you add, the higher the chance of false positives, but it never falsely claims an item is absent.


## Code Sample
```golang
package main
import (
    "fmt"
    "hash/fnv"
)
type BloomFilter struct {
    bitArray []bool
    size     int
    hashFuncs []func(string) int
}
func NewBloomFilter(size int, hashFuncs []func(string) int) *BloomFilter {
    return &BloomFilter{
        bitArray: make([]bool, size),
        size:     size,
        hashFuncs: hashFuncs,
    }
}

func (bf *BloomFilter) Add(item string) {
    for _, hashFunc := range bf.hashFuncs {
        index := hashFunc(item) % bf.size
        bf.bitArray[index] = true
    }
}

func (bf *BloomFilter) MightContain(item string) bool {
    for _, hashFunc := range bf.hashFuncs {
        index := hashFunc(item) % bf.size
        if !bf.bitArray[index] {
            return false
        }
    }
    return true
}
func main() {
    // Example hash function
    hashFunc := func(s string) int {
        h := fnv.New32a()
        h.Write([]byte(s))
        return int(h.Sum32())
    }

    // Create a Bloom filter with size 100 and one hash function
    bloom := NewBloomFilter(100, []func(string) int{hashFunc})

    // Add items
    bloom.Add("apple")
    bloom.Add("banana")

    // Check for existence
    fmt.Println(bloom.MightContain("apple"))  // true
    fmt.Println(bloom.MightContain("banana")) // true
    fmt.Println(bloom.MightContain("cherry")) // false (might be false positive)
}
```
## Constraints
- **Space Efficiency**: Bloom filters use a fixed amount of space, regardless of the number of items added. The size of the bit array and the number of hash functions determine the false positive rate.
- **False Positive Rate**: The more items you add, the higher the chance of false positives. Variations like Scalable Bloom Filters can help manage false positives by dynamically adjusting the size of the filter.
- **No Deletion**: Once an item is added, it cannot be removed from a standard Bloom filter. Variations like Counting Bloom Filters allow for deletions but at the cost of additional complexity and space.

## Variations of bloom filter
- **Counting Bloom Filter**: Allows for item deletion by using a counter for each index of the array.
- **Scalable Bloom Filter**: Dynamically adjusts its size as more items are added, maintaining a low false positive rate.

## Use cases
- **Web Caching**: Quickly check if a URL is in the cache. If the Bloom filter says it’s not, you can skip the cache lookup entirely.
- **Distributed Systems**: In systems like Apache Cassandra and Amazon DynamoDB, Bloom filters are used to avoid unnecessary disk reads for non-existent keys.
- **Blockchain**: Nodes use Bloom filters to filter transactions, allowing them to quickly determine if they have seen a transaction without storing every transaction.
- **Search Engines**: Quickly check if a URL has been indexed, reducing the need for full scans of the index.
- **Database Queries**: Reduce disk lookups by checking for existence in a Bloom filter first. 
- **Network Security**: Identify known malicious IP addresses without storing all of them.