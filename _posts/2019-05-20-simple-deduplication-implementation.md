---
layout: post
title:  "Simple Deduplication (Implementation)"
image: ''
date:   2019-05-20 00:12:00
tags:
- Deduplication
- Java
description: 'Storage technique to efficiently store data in cloud'
categories:
- Deduplication
---

## Structure

I've made the implementation in total of 4 .java files

* **Deduplication** main process, all the chunking, fingerprinting, indexing is done here
* **Chunk** single chunk handler
* **Index** in order to utilize ***Serializable***, interface is required, which is dealt here
* **Storage** handles storage related procedures (i.e. create, get, etc.)

The program will be able to execute 3 commands

* **Upload**
* **Download**
* **Delete**

For **Upload** it will require more than just the file. It requires

* min = Minimum chunk size (in bytes)
* avg = Average chunk size (in bytes)
* max = Maximum chunk size (in bytes)
* d = Base parameter
* dir = File location

For **Download** it will only require the original file name and location to save. For **Delete** it will only require the original file name. Note that the filename before deduplication is the original filename that will be used to reference the deduplicated data.

The index file will be stored in the same directory that the programming is running on and the data will be stored in a data folder which will be automatically created on the first deduplication.

---

## Code Snippets

Because the whole program is kind of big for a blog post, I decided to only put in the main Chunking, Fingerprinting, Indexing portion only. Sorry for the messy variable names.

* **m** minimum size
* **q** average size
* **x** maximum size
* **d** base parameter

**Checking if the Chunk is New (i.e. Unique)** Notice Fingerprinting is processed here

{% highlight java %}
if(chunkFound == false) {
  if(size >= request.m) {
    if(size == request.m) {
      for(int i = 1; i <= request.m; i++) {
        int t = byteStore.get(i - 1);
        int e = request.m - i;
        int mod = (int) modExpOpt(request.d, e, request.q);
        rfp += (t * mod) % request.q;
      }
      rfp = rfp % request.q;
    } else if(size > request.m) {
      int lastByte = byteStore.get(size - request.m);
      int q = request.q;
      int d = (request.d % request.q);
      int psMinus1 = (lastRfp % request.q);
      int d_mMinus1 = (preMod % request.q);
      int ts = (lastByte % request.q);
      int tsPlusM = (data % request.q);

      int tempPart1 = ((d_mMinus1 % q) * (ts % q)) % q;
      int tempPart2 = ((psMinus1 % q) - tempPart1) % q;
      rfp = ((((d % q) * (tempPart2)) % q) + (tsPlusM % q)) % q;
      if(rfp < 0) {
        rfp += request.q;
      }
    }
    if(rfp == 1) {
      offsets.add(offset);
      chunkFound = true;
    }
  }
  lastRfp = rfp;
{% endhighlight %}

**If Chunk does not exist in Index** (note that "SHA-256" was used to encrypt data)

{% highlight java %}
if(chunkFound) {
  byte[] chunk = Arrays.copyOfRange(byteStore.array(), 0, size);
  MessageDigest md = MessageDigest.getInstance("SHA-256");
  md.update(chunk, 0, size);
  byte[] checksumInByte = md.digest();
  String checksum = new BigInteger(1, checksumInByte).toString(16);
  Chunk ch = index.chunks.get(checksum);

  if (ch == null) {
    ch = new Chunk(1);
    storage.put(checksum, new Chunk(chunk));
    bytesUploaded += size;
    chunksUploaded++;
    localChunks.put(checksum, 0);
  } else {
    if(localChunks.get(checksum) == null){
      localChunks.put(checksum, 0);
      ch.increment();
    }
  }

  index.chunks.put(checksum, ch);
  chunks.add(checksum);
  byteStore.clear();
  size = 0;
  chunkFound = false;
}
{% endhighlight %}

---

## Tests and Results

For test, I've downloaded the plain text file of "Tale of Two Cities" from gutenberg.org.

The first upload is the original file.

{% highlight text %}
$ java Deduplication upload 128 256 512 1 tale-of-two-cities-1.txt
-------------------- Upload Report --------------------
Chunks: 2343
Unique Chunks: 2343
Total Bytes Deduplicated: 787075
Total Bytes Not Deduplicated: 787075
Duplication  Ratio: 0.00%
-------------------- End Upload Report --------------------
{% endhighlight %}

As seen in the output above, all of the chunks are unique since this is the first file every uploaded. For the second upload, I've deleted few of the paragraphs in the text file so that the file is not exactly the same.

{% highlight text %}
$ java Deduplication upload 128 256 512 1 tale-of-two-cities-2.txt
-------------------- Upload Report --------------------
Chunks: 2345
Unique Chunks: 3
Total Bytes Deduplicated: 1073
Total Bytes Not Deduplicated: 788059
Duplication  Ratio: 99.86%
-------------------- End Upload Report --------------------
{% endhighlight %}

Now since most of the file is exactly the same, it has lots of duplicated chunks from the first upload. Basically I have just uploaded 2 files, but I have reduced the storage compared to storing both files!

Time to move on to downloading the modified file.

{% highlight text %}
$ java Deduplication download tale-of-two-cities-2.txt tale-of-two-cities-2-copy.txt
-------------------- Download Report --------------------
Chunks Downloaded: 2345
Total Bytes Downloaded: 789132
Total Bytes Reconstructed: 788059
-------------------- End Download Report --------------------
{% endhighlight %}

Reconstructed bytes are the sum of bytes that are not unique to this file. Basically these bytes are shared chunks. Now let's try deleting the modified file.

{% highlight text %}
$ java Deduplication delete tale-of-two-cities-2.txt
-------------------- Delete Report --------------------
Chunks Deleted: 3
Bytes Deleted: 1073
-------------------- End Download Report --------------------
{% endhighlight %}

The numbers are very similar when I have uploaded the modified file. The other chunks are needed for the original files so they are still stored. But, what if I didn't delete the modified file and deleted the original file instead?

{% highlight text %}
$ java Deduplication delete tale-of-two-cities.txt
-------------------- Delete Report --------------------
Chunks Deleted: 1
Bytes Deleted: 128
-------------------- End Download Report --------------------
{% endhighlight %}

Because only 1 chunk is not shared with the modified file, it is now deleting only 1 chunk that's in the size of 89 bytes. Note that because the minimum size byte set to 128, it says 128, but the actual real unique file byte size is 89 (rest are filled up with zeroes).

---

## External Libraries Used

* common-lang3-3.1.jar (from Apache Common)
