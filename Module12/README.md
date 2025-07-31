# Team Activity: Explore Hash Algorithms and Collisions  

For this team activity, you will explore hashing algorithms and collisions. When working with Hashmaps,
there will be collisions, but the depth of the lists (or even trees as you will explore in future weeks),
is often very small. This is why even with large numbers, such as near a million items, you still get effectively
$O(1)$ access time. 

## Review The Code

In your group, review the provided code in [hashing.h](hashing.h) and [hash_test.c](hash_test.c). 
Explain to each other the various lines. `hash_test.c` probably has code that looks more familiar to you.
The code that may look unique (though used in past test files), is the code that includes function pointers.

```c
uint32_t (*hash[])(char*) = {
    simple_hash,
    djb2,
    fnv_hash,
    jenkins_one_at_a_time_hash    
};
```

What this array stores is a pointer to each function, but also defines that the functions will return an `uint32_t` and are 
required to take in a `char *` (string) as a parameter. If you want to call `simple_hash` it would be `simple_hash[0]`.

Later in your code, you will see:

```c
for (int i = 0; i < ALGO_SIZE; i++)
{
      uint32_t loc = (hash[i](buffer)) % SIZE;
      collisions[i][loc]++;
}
```
This means that for every algorithm, call the hash, mod it by the `SIZE` of the 'hashmap' (really just an array in this case), and increment a counter to track the number of collisions that happen. Since the `collisions[i]` is initialized with 0, it will just continue to count up for everything a key wants to be stored in that position. 

Then later in `void printCollisionsOnly(int *array, int size)` it prints out the stats of the collisions array for each algorithm in order. 

This code is setup so you can get a better feel of how hashmaps work in practice, even if we are not storing the full key-value pairs. 



### *.txt files
There are a number of .txt files. If you open them you will find the files with ids in the name contain values such as `tt0000001`, where as the titles contain the corresponding movie titles, like the `The Princess Bride` which is on line 53,321. The ids and titles are pulled from the open IMDB dataset, and filtered for US only movies with IMDB ratings to reduce it down from the 32 million points to only 925,174 titles. Then using `head -n #` smaller test files were developed at 1000, 10,000, and 100,000. 

### Running the Code
You should be able to run the code with:

```console
> clang -Wall hash.test.c 
> ./a.out movie_ids_us_1000.txt
```

You can also run more than one file at a time chaining the results:

```console
> ./a.out movie_ids_us_1000.txt movie_titles_us_1000.txt movie_ids_us_10000.txt movie_titles_us_10000.txt movie_ids_us_100000.txt movie_titles_us_100000.txt movie_ids_us_unique.txt movie_titles_us_unique.txt 
```
The above is all one line, but auto wraps in display. 

You should go ahead and run the code. When running it, you will see something like (depending on how many files you test against):

```text
movie_ids_us_1000.txt...
Collisions: 1000, Highest: 1000, Average Length > 1: 1000.00, Filled Spots: 1, Load: 0.00000
Collisions: 1000, Highest: 1000, Average Length > 1: 1000.00, Filled Spots: 1, Load: 0.00000
Collisions: 0, Highest: 0, Average Length > 1: -nan, Filled Spots: 1000, Load: 0.00050
Collisions: 0, Highest: 0, Average Length > 1: -nan, Filled Spots: 1000, Load: 0.00050

movie_titles_us_1000.txt...
Collisions: 1000, Highest: 1000, Average Length > 1: 1000.00, Filled Spots: 1, Load: 0.00000
Collisions: 1000, Highest: 1000, Average Length > 1: 1000.00, Filled Spots: 1, Load: 0.00000
Collisions: 22, Highest: 2, Average Length > 1: 2.00, Filled Spots: 989, Load: 0.00049
Collisions: 22, Highest: 2, Average Length > 1: 2.00, Filled Spots: 989, Load: 0.00049
```
The first two lines are high because those functions are not implemented yet, so only returning 0. 

> Discussion: What does each value represent out of Collisions, Highest, Average Length > 1, Filled Spots, and Load. Make sure to discuss together what is the ideal load balance num.
> * Total - number of collisions that occured (when array[i] > 1)
> * Load  - the fraction of table occupied (counter/SIZE)
> * Filled - number of spots occupied
> * Highest - Spot with the most collisions (longest collision chain)
> * Average Length > 1 - Average length of collisions (total/length)


## 👉🏽 **Task**: Write Simple Hash

Using your notes from the class videos or on your own, write a simple hash that that adds the ASCII value of the characters. As a reminder, just casting a character as an int gives you the ascii value, so you can loop through every character in the string, and
just add the values of those characters.

> Discussion: How effective is the "simple hash" compared to the other two hashes that are implemented - for this particular dataset"? 
> it is so bad

## 👉🏽 **Task**: Write djb2 Hash

Now work on the djb2 hash, which as a reminder, is:

```text
hash = 5381
for c in string:
    hash = hash * 33 + c
```

We encourage you to look at a more efficient implementation than using multiplication.
* We could use a bitwise shift (shift operator) instead of multiplication.

> Discussion: How effective is the djb2 compared to others for this particular dataset? 
>  Similar performance

> Second: What is a more efficient implementation? Do you think they picked 33 for this reason?
> Dan Bernstein tested a bunch of different numbers, and 33 proved to be best based on his empirical results. There are a variety of mathematical reasons behind this, and we understand some of them! See below...
> `hash = ((hash << 5) + hash) + c; // hash * 33 + c`
> 33 is an odd number that is represented with just 2 bits being set (the absolute minimum for an odd), 
> which means many optimizations can be made to algorithms to speed them up. 
> In this case, very expensive multiplications can be replaced with a single-cycle shift & add. 
> (Multiplications are much faster now in 2021 than they were in 1991 when the algorithm was first designed) 
> https://www.reddit.com/r/3Blue1Brown/comments/nljzi8/magic_number_33_in_djb2_hash_algorithm/

> Why would this matter? (reflect back to your assembly team activity).
> The number we chose for the hash function affects the number of collisions and speed. It makes sense to choose an optimal number rather than a random one (at least in this case, may change based on program/data names). So why 33? Judson knows! 

> In assembly, multiplication is more expensive than addition and shifting. By picking 33, you can replace a multiply with a shift and add, making the hash function faster and more suitable for performance-critical code (like in hash tables).

## Tests

Look at the file, and run various tests.
# Hash Table Performance Statistics

| File | Test Run | Collisions | Highest | Avg Length > 1 | Filled Spots | Load |
|------|----------|------------|---------|----------------|--------------|------|
| movie_ids_us_1000.txt | 1 | 999 | 81 | 38.42 | 27 | 0.00003 |
| movie_ids_us_1000.txt | 2 | 0 | 0 | -nan | 1000 | 0.00100 |
| movie_ids_us_1000.txt | 3 | 0 | 0 | -nan | 1000 | 0.00100 |
| movie_ids_us_1000.txt | 4 | 0 | 0 | -nan | 1000 | 0.00100 |
| movie_titles_us_1000.txt | 1 | 391 | 5 | 2.22 | 785 | 0.00078 |
| movie_titles_us_1000.txt | 2 | 22 | 2 | 2.00 | 989 | 0.00099 |
| movie_titles_us_1000.txt | 3 | 22 | 2 | 2.00 | 989 | 0.00099 |
| movie_titles_us_1000.txt | 4 | 22 | 2 | 2.00 | 989 | 0.00099 |
| movie_ids_us_10000.txt | 1 | 9999 | 697 | 285.69 | 36 | 0.00004 |
| movie_ids_us_10000.txt | 2 | 142 | 2 | 2.00 | 9929 | 0.00993 |
| movie_ids_us_10000.txt | 3 | 60 | 2 | 2.00 | 9970 | 0.00997 |
| movie_ids_us_10000.txt | 4 | 114 | 2 | 2.00 | 9943 | 0.00994 |
| movie_titles_us_10000.txt | 1 | 9289 | 22 | 5.51 | 2398 | 0.00240 |
| movie_titles_us_10000.txt | 2 | 853 | 4 | 2.13 | 9547 | 0.00955 |
| movie_titles_us_10000.txt | 3 | 866 | 4 | 2.13 | 9541 | 0.00954 |
| movie_titles_us_10000.txt | 4 | 873 | 4 | 2.13 | 9537 | 0.00954 |
| movie_ids_us_100000.txt | 1 | 99999 | 6029 | 2222.20 | 46 | 0.00005 |
| movie_ids_us_100000.txt | 2 | 5528 | 3 | 2.01 | 97220 | 0.09722 |
| movie_ids_us_100000.txt | 3 | 10674 | 4 | 2.05 | 94538 | 0.09454 |
| movie_ids_us_100000.txt | 4 | 9346 | 4 | 2.03 | 95250 | 0.09525 |
| movie_titles_us_100000.txt | 1 | 99124 | 144 | 28.63 | 4338 | 0.00434 |
| movie_titles_us_100000.txt | 2 | 25197 | 17 | 2.43 | 85167 | 0.08517 |
| movie_titles_us_100000.txt | 3 | 25195 | 17 | 2.43 | 85183 | 0.08518 |
| movie_titles_us_100000.txt | 4 | 25210 | 17 | 2.43 | 85146 | 0.08515 |
| movie_ids_us_unique.txt | 1 | 924735 | 38290 | 8723.92 | 107 | 0.00011 |
| movie_ids_us_unique.txt | 2 | 569421 | 8 | 2.40 | 592710 | 0.59271 |
| movie_ids_us_unique.txt | 3 | 557431 | 9 | 2.36 | 603710 | 0.60371 |
| movie_ids_us_unique.txt | 4 | 558694 | 8 | 2.36 | 602868 | 0.60287 |
| movie_titles_us_unique.txt | 1 | 924092 | 1082 | 129.12 | 8240 | 0.00824 |
| movie_titles_us_unique.txt | 2 | 603672 | 200 | 3.07 | 518084 | 0.51808 |
| movie_titles_us_unique.txt | 3 | 603876 | 200 | 3.07 | 517979 | 0.51798 |
| movie_titles_us_unique.txt | 4 | 603322 | 201 | 3.07 | 518217 | 0.51822 |

* What happens when you change SIZE?
  * The load changes, usually increasing with greater size
  * Highest chain length changes, usually increases with greater size
  * 

* Is there an optimal number/size, that increases your load factor but not too much (0.5-0.7)?
  * it looks like the load factor approaches ideal size for the largest (~900000) datasets

* What are the differences between the various functions?
  * Number of operations?
    * simple -> 1 addition per character
    * DJB2 -> 1 multiplication (or shift+add) and 1 addition per character
    * FNV -> 1 XOR and 1 multiplication per character
    * One at a time -> Several additions, shifts, and XORs per character, plus some final mixing steps.
  * Types of operations?
    * simple -> Multiplication & addition
    * DJB2 -> Multiplication
    * FNV -> XOR and multiplication
    * One at a time -> Addition, XOR, Shift
  * Do the ones that perform better come at an increased cost?
    * No! 
    * Simple -> Technically the fastest, but
   
    * DJB2 -> Slightly more expensive than simple_hash, but much better distribution. Using shift+add makes it nearly as fast as simple_hash.
    * FNV ->  Multiplication is a bit more expensive than addition, but still fast. Good distribution.
    * One at a time -> Most expensive per character, but provides very good distribution and mixing.

* Would prime numbers be useful staring points?
  * 

If you don't recall what some operators do - look them up! Discuss them (particular the XOR and the shifts). Why would these be useful?
* 

As a reminder, hash algorithms care more about the binary bits, which thinking about it in a form of binary may help better understand why they may use those. 

For example, take 5 in binary (101) and apply the operations to that small value, to get a better idea of the transformations that are happening. You can always either write a small see program, or perform the same operations in the python interactive interrupter to see what is going on! 

```python
>>>bin(5)
'0b101'
>>>bin(5<<2) 
'0b10100'
>>>bin(5*4)
'0b10100'
```

## Final Discussion

You will never find a perfect hash, and there is often a cost of performance the more complicated your transformation. As such, those who define hashmaps are often searching for a "good enough" hash algorithm, and then they provide a way to let people change the default implementation so they can be more specific for their domain. 

For object oriented languages such as Java, those are built into the objects themselves to determine their hash. 

> Discussion: 
> Discuss a hashmap design. See if you can explain to each other how it is designed, and why it is effectively $O(1)$.  
> Can you also generate a case where access/insert/delete are $O(n)$?

## Leet Code Practice
Take time practicing some of the past modules leet code. While you may not have time for everyone to do this, have a couple people practice "live coding". Live coding is a skill in interviews were you are asked to describe code **while** you are writing it. It can be a challenging skill, and it takes practice. I recommend that you setup a rotation of people to practice this skill within your team, ideally a couple every week. The other teams members can offer support, and then do a code review after a solution is generated. Discuss pros/cons of the leet code solution as well as other potential ways to solve the problem.

## 📚 Resources
* [Left/Right Shift Operators in C](https://www.geeksforgeeks.org/left-shift-right-shift-operators-c-cpp/)
* [List of Hash algorithms](https://en.wikipedia.org/wiki/List_of_hash_functions#Non-cryptographic_hash_functions)
* [djb2 Hash explained](https://theartincode.stanis.me/008-djb2/)
