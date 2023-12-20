# Huffman table hacking tool



This tool calculates tricky canonical huffman histogram, which is able to trigger OOB (Out Of Band) write for vulnerable libwebp library (i.e. libwebp <= `1.3.1`). This vulnerability is known as `CVE-2023-4863` or `CVE-2023-41064`. We can overflow the pre-allocated huffman table by  at most **132 entries**.

The rationale is that libwebp assumes the huffman histogram contained in each webp image is well formated. That is, the decoding tree must be a **complete tree**. With this assumption, the libwebp group took advantage of an [enough](https://github.com/madler/zlib/blob/develop/examples/enough.c) tool to estimate the maximum memory size to build the decoding huffman table (a powerful structure to decode canonical huffman codes rapidly). However, hackers are able to construct an **incomplete** tree to go beyond this memory limit and thus overflow the pre-allocated memory.

This hacking tool was created by adjusting [enough](https://github.com/madler/zlib/blob/develop/examples/enough.c) with incomplete tree support.


Compile the code using command below:


```bash
gcc -o NotEnough ./main.c
```

For a huffman table with 40-symbol, 8-bit root table and maximum depth being 15, the maximum number of table entries is `410`. However, by constructing an incomplement tree, the number of table entries could be `542`. Try the command below:


```bash
./NotEnough 40 8 15
```

An example of such trees is as below:


![Bad tree](img/graphviz-40-542-tree.svg)

Note that we pruned the branches with no leaves for brief.

 You can then use the [craft](https://github.com/mistymntncop/CVE-2023-4863) tool to construct an effective webp image to overflow **dwebp** tool (version <= `1.3.1`). Change the huffman code histogram **code\_lengths\_counts[4]** to {0, 1, 0, 0, 0, 0, 0, 0, 0, 3, 5, 9, 17, 1, 1, 3} accordingly, and rebuild the tool:



```bash
vim craft.c    # Change huffman code histogram accordingly, i.e. about line 495.
gcc -o craft craft.c
./craft -o bad_542.webp
```



Please check [@benhawkes's blog](https://blog.isosceles.com/the-webp-0day/) to have a better idea of `CVE-2023-4863`.



