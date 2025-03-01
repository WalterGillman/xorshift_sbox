# Xorshift+Rijndael S-BOX

This source implements the xorshift algorithm as described in [1], specifically the version with 4 state words with each word being 8 bits for a 2^32-1 period.

The xorshift part uses the mininum triplet 1,1,3 that is also used by other sources and the algorithm in the paper adapted to 8 bits.

After the xorshift step, the seed is passed through a series of Rijndael s-boxes[2] using a table lookup to produce an 8-bit pseudo-random number.

Note that the first two xorshifts and the last xorshift can be implemented as a table lookup as well so other triplets need not be less desirable in 8 bit cpus if memory is not an issue.

The C version passes the PractRand tests(stdin) for the whole period.

```
RNG_test using PractRand version 0.94
RNG = RNG_stdin, seed = 1
test set = core, folding = standard(unknown format)

rng=RNG_stdin, seed=1
length= 4 gigabytes (2^32 bytes), time= 37.0 seconds
  no anomalies in 282 test result(s)
```

Usual limitations of xorshift apply(seed cannot be zero).

Included also are the following ZX USR routines in z80 assembly:
* xorshift_sbox.asm: A reference implementation(clobbers I)
* xorshift_sbox_8.asm: A more optimized version 
* xorshift_sbox_16.asm: A more optimized version returning 16 bits(clobbers I); including a seeding and a uniform random number function.

The 16 bit version can be used as a drop-in replacement for RND in BASIC

```
10 DEF FN r()=(USR 32772)/65536
20 PRINT FN r()
```

It is also possible to use the void seeding function T(a) for RANDOMIZE and the uniform random-integer-returning U(a) function for RND*(a+1) which is a common sight in BASIC programs but produces biased pseudo-random numbers.

```
9010 DEF FN T(a)=USR 32864
9010 DEF FN U(a)=USR 32880
```

## Description

The idea behind the algorithm is to add a scrambling step to the output of the xorshift algorithm which, even though performant in CPUs lacking variable shift and 32-bit operations, compares poorly to even 32-bit xorshift truncated to 8 bits in terms of output quality.

### Version 1

```
S(w ⊕ S( z ⊕ S( y ⊕ S(x))))
```

By passing a word through the S-box, the word aquires a value which is linearly unrelated to the next. In this way, we can xor them whithout suffering from the linear relationships between them.

Ideally, we would want to achieve an avalanche effect, and indeed we can see that flipping a state bit produces an output with each bit flipped with a probability between 40 to 60%. The last words in the pipeline have a larger and more variable influence that averages closer to 50%.

The version presented here also passes TestU01 SmallCrush test suite(other versions and variations have not been tested nor have been other test suites).

### Version 2

It is possible to apply shorter transformations for output that also passes PractRand.

```
w ⊕ S( z ⊕ y ⊕ S(x ⊕ w))))
```

We lose some of the effect of the S-box by xoring related values before performing the step but the original algorithm has good-enough statistical properties that this suffices to produce a passing result.

In this version, there is little variation on how each state affects the output with the range being again within the 40 to 60% range.

The scrambling step described above can be applied to other families of weak PRNGs such as LCGs to make their output pass all PractRand tests. This shorter version, allows lcg8of32sbox(a=1664525,c=1013904223) to both pass and to perform comparably(about 20% slower) to sfc32 when embedded in PractRand.
The actual performance of each variant will vary depending on the particular CPU architecture, acknowledging that even on an optimal architecture, there is a fundamental limitation in having to compute the results serially and on the whole word, and the relative size of the ouput word will decrease as that of the state increases.

### Version 3

The scrambling from the previous formulae is however not enough to make a counter pass the PractRand tests. This can be achieved using the following formula.


```
x ⊕ y ⊕ z ⊕ w ⊕ S(w ⊕ S(x ⊕ S(y ⊕ S(z ⊕ S(w ⊕ S( z ⊕ S( y ⊕ S(x))))))))

```

As more steps are added, it is less easy to reason on what is happening to the state. Longer chains with near-perfect statistical results on the avalanche tests actually consistently get a suspicious result at 4GB indicating that they either introduce or reveal a pattern in the state perhaps related to the short cycles in the S-box.

## Bibliography

[1] Marsaglia, George. (2003). Xorshift RNGs. Journal of Statistical Software. 08. 10.18637/jss.v008.i14.

[2] Dworkin, M.J., Barker, E.B., Nechvatal, J., Foti, J., Bassham, L.E., Roback, E.A., & Dray, J.F. (2001). Advanced Encryption Standard (AES).

## Contact

contact: waltergillman@proton.me
monero:44VDKbaBENcQvDQzY3Rjf2UzquTbzn4CLb96wEwDuVLPXuLT8YvVwD299T6VEDTCrbimpfh5Ke7YuLmqzAgSFCD4GWa8Kj4
