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
* xorshift_sbox_16.asm: A more optimized version returning 16 bits(clobbers I)

The 16 bit version can be used as a drop-in replacement for RND in BASIC

```
10 DEF FN r()=(USR 32768)/65536
20 PRINT FN r()
```

[1] Marsaglia, George. (2003). Xorshift RNGs. Journal of Statistical Software. 08. 10.18637/jss.v008.i14.

[2] Dworkin, M.J., Barker, E.B., Nechvatal, J., Foti, J., Bassham, L.E., Roback, E.A., & Dray, J.F. (2001). Advanced Encryption Standard (AES).

contact: waltergillman@proton.me
monero:44VDKbaBENcQvDQzY3Rjf2UzquTbzn4CLb96wEwDuVLPXuLT8YvVwD299T6VEDTCrbimpfh5Ke7YuLmqzAgSFCD4GWa8Kj4
