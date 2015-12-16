<!-- title: Test Data -->

Test Data ... for your viewing pleasure
=======================================

Two data sets follow: *valid records* and *invalid records*.
Explanations of each case will be succint. Enjoy!


Valid Records
-------------

### Standard Bulletin 1
    Case title : SB1
    Case description :
        (1) valid bulletin
        (2) less than 80 bytes in total length
        (3) no location datum

    Type    : Bulletin
    Message : Hello world!
    Time    : 12345678
    
    Results~
    Ombuds Header : 
    4f 4d 42 55 44 53 01 14

    Ombuds WireRecord :
    0a 0c 48 65 6c 6c 6f 20 77 6f 72 6c 64 21 10 ce c2 f1 05

    Formats~
    P2PKH :
    [0] 76 a9 14 4f 4d 42 55 44 53 01 14 0a 0c 48 65 6c 6c 6f 20 77 6f 72 6c 88 ac
    [1] 76 a9 14 64 21 10 ce c2 f1 05 00 00 00 00 00 00 00 00 00 00 00 00 00 88 ac

    Null Data :
    [0] 6a 1b 4f 4d 42 55 44 53 01 14 0a 0c 48 65 6c 6c 6f 20 77 6f 72 6c 64 21 10
        ce c2 f1 05


### Standard Bulletin 2
    Case title : SB2
    Case description :
        (1) valid bulletin
        (2) less than 80 bytes in total length
        (3) includes location datum

    Type    : Bulletin
    Message : Hello world! 
    Time    : 12345678
    Loc     : 38.8977, 77.0366, 1000
    
    Results~
    Ombuds Header : 
    4f 4d 42 55 44 53 01 31

    Ombuds WireRecord :
    0a 0c 48 65 6c 6c 6f 20 77 6f 72 6c 64 21 10 ce c2 f1 05 1a 1b 09 42 cf 66 d5 
    e7 72 43 40 11 27 c2 86 a7 57 42 53 40 19 00 00 00 00 00 40 8f 40

    Formats~
    P2PKH :
    [0] 76 a9 14 4f 4d 42 55 44 53 01 31 0a 0c 48 65 6c 6c 6f 20 77 6f 72 6c 88 ac
    [1] 76 a9 14 64 21 10 ce c2 f1 05 1a 1b 09 42 cf 66 d5 e7 72 43 40 11 27 88 ac
    [2] 76 a9 14 c2 86 a7 57 42 53 40 19 00 00 00 00 00 40 8f 40 00 00 00 00 88 ac

    Null Data :
    [0] 38 4f 4d 42 55 44 53 01 31 0a 0c 48 65 6c 6c 6f 20 77 6f 72 6c 64 21 10 ce 
        c2 f1 05 1a 1b 09 42 cf 66 d5 e7 72 43 40 11 27 c2 86 a7 57 42 53 40 19 00 
        00 00 00 00 40 8f 40

### Standard Bulletin 3
    Case title : SB3
    Case description :
        (1) valid bulletin
        (2) greater than 80 bytes in total length
        (3) no location datum

    Type    : Bulletin
    Message : When in the Course of human events, it becomes necessary for one 
              people to dissolve the political bands which have connected them 
              with another, and to assume among the powers of the earth, the 
              separate and equal station to which the Laws of Nature and of 
              Nature's God entitle them, a decent respect to the opinions of 
              mankind requires that they should declare the causes which impel 
              them to the separation.
    Time    : 1234567890 

    Results~
    Ombuds Header : 
    4f 4d 42 55 44 53 01 fd a0 01

    Ombuds WireRecord :
    0a 96 03 57 68 65 6e 20 69 6e 20 74 68 65 20 43 6f 75 72 73 65 20 6f 66 20 68 
    75 6d 61 6e 20 65 76 65 6e 74 73 2c 20 69 74 20 62 65 63 6f 6d 65 73 20 6e 65 
    63 65 73 73 61 72 79 20 66 6f 72 20 6f 6e 65 20 70 65 6f 70 6c 65 20 74 6f 20 
    64 69 73 73 6f 6c 76 65 20 74 68 65 20 70 6f 6c 69 74 69 63 61 6c 20 62 61 6e 
    64 73 20 77 68 69 63 68 20 68 61 76 65 20 63 6f 6e 6e 65 63 74 65 64 20 74 68 
    65 6d 20 77 69 74 68 20 61 6e 6f 74 68 65 72 2c 20 61 6e 64 20 74 6f 20 61 73 
    73 75 6d 65 20 61 6d 6f 6e 67 20 74 68 65 20 70 6f 77 65 72 73 20 6f 66 20 74 
    68 65 20 65 61 72 74 68 2c 20 74 68 65 20 73 65 70 61 72 61 74 65 20 61 6e 64 
    20 65 71 75 61 6c 20 73 74 61 74 69 6f 6e 20 74 6f 20 77 68 69 63 68 20 74 68 
    65 20 4c 61 77 73 20 6f 66 20 4e 61 74 75 72 65 20 61 6e 64 20 6f 66 20 4e 61 
    74 75 72 65 27 73 20 47 6f 64 20 65 6e 74 69 74 6c 65 20 74 68 65 6d 2c 20 61 
    20 64 65 63 65 6e 74 20 72 65 73 70 65 63 74 20 74 6f 20 74 68 65 20 6f 70 69 
    6e 69 6f 6e 73 20 6f 66 20 6d 61 6e 6b 69 6e 64 20 72 65 71 75 69 72 65 73 20 
    74 68 61 74 20 74 68 65 79 20 73 68 6f 75 6c 64 20 64 65 63 6c 61 72 65 20 74 
    68 65 20 63 61 75 73 65 73 20 77 68 69 63 68 20 69 6d 70 65 6c 20 74 68 65 6d 
    20 74 6f 20 74 68 65 20 73 65 70 61 72 61 74 69 6f 6e 2e 10 d2 85 d8 cc 04

    Formats~
    P2PKH :
    [0] 76 a9 14 4f 4d 42 55 44 53 01 fd a0 01 0a 96 03 57 68 65 6e 20 69 6e 88 ac
    [1] 76 a9 14 20 74 68 65 20 43 6f 75 72 73 65 20 6f 66 20 68 75 6d 61 6e 88 ac
    [2] 76 a9 14 20 65 76 65 6e 74 73 2c 20 69 74 20 62 65 63 6f 6d 65 73 20 88 ac
    [3] 76 a9 14 6e 65 63 65 73 73 61 72 79 20 66 6f 72 20 6f 6e 65 20 70 65 88 ac
    [4] 76 a9 14 6f 70 6c 65 20 74 6f 20 64 69 73 73 6f 6c 76 65 20 74 68 65 88 ac
    [5] 76 a9 14 20 70 6f 6c 69 74 69 63 61 6c 20 62 61 6e 64 73 20 77 68 69 88 ac
    [6] 76 a9 14 63 68 20 68 61 76 65 20 63 6f 6e 6e 65 63 74 65 64 20 74 68 88 ac
    [7] 76 a9 14 65 6d 20 77 69 74 68 20 61 6e 6f 74 68 65 72 2c 20 61 6e 64 88 ac
    [8] 76 a9 14 20 74 6f 20 61 73 73 75 6d 65 20 61 6d 6f 6e 67 20 74 68 65 88 ac
    [9] 76 a9 14 20 70 6f 77 65 72 73 20 6f 66 20 74 68 65 20 65 61 72 74 68 88 ac
    [10] 76 a9 14 2c 20 74 68 65 20 73 65 70 61 72 61 74 65 20 61 6e 64 20 65 88 ac
    [11] 76 a9 14 71 75 61 6c 20 73 74 61 74 69 6f 6e 20 74 6f 20 77 68 69 63 88 ac
    [12] 76 a9 14 68 20 74 68 65 20 4c 61 77 73 20 6f 66 20 4e 61 74 75 72 65 88 ac
    [13] 76 a9 14 20 61 6e 64 20 6f 66 20 4e 61 74 75 72 65 27 73 20 47 6f 64 88 ac
    [14] 76 a9 14 20 65 6e 74 69 74 6c 65 20 74 68 65 6d 2c 20 61 20 64 65 63 88 ac
    [15] 76 a9 14 65 6e 74 20 72 65 73 70 65 63 74 20 74 6f 20 74 68 65 20 6f 88 ac
    [16] 76 a9 14 70 69 6e 69 6f 6e 73 20 6f 66 20 6d 61 6e 6b 69 6e 64 20 72 88 ac
    [17] 76 a9 14 65 71 75 69 72 65 73 20 74 68 61 74 20 74 68 65 79 20 73 68 88 ac
    [18] 76 a9 14 6f 75 6c 64 20 64 65 63 6c 61 72 65 20 74 68 65 20 63 61 75 88 ac
    [19] 76 a9 14 73 65 73 20 77 68 69 63 68 20 69 6d 70 65 6c 20 74 68 65 6d 88 ac
    [20] 76 a9 14 20 74 6f 20 74 68 65 20 73 65 70 61 72 61 74 69 6f 6e 2e 10 88 ac
    [21] 76 a9 14 d2 85 d8 cc 04 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 88 ac

    Null Data :
    FAIL

### Standard Bulletin 4
    Case title : SB4
    Case description :
        (1) valid bulletin
        (2) greater than 80 bytes in total length
        (3) includes location datum

    Type    : Bulletin
    Message : When in the Course of human events, it becomes necessary for one 
              people to dissolve the political bands which have connected them 
              with another, and to assume among the powers of the earth, the 
              separate and equal station to which the Laws of Nature and of 
              Nature's God entitle them, a decent respect to the opinions of 
              mankind requires that they should declare the causes which impel 
              them to the separation.
    Time    : 1234567890 
    Loc     : 38.8977, 77.0366, 1000

    Results~
    Ombuds Header : 
    4f 4d 42 55 44 53 01 fd bd 01

    Ombuds WireRecord :
    0a 96 03 57 68 65 6e 20 69 6e 20 74 68 65 20 43 6f 75 72 73 65 20 6f 66 20 68 
    75 6d 61 6e 20 65 76 65 6e 74 73 2c 20 69 74 20 62 65 63 6f 6d 65 73 20 6e 65 
    63 65 73 73 61 72 79 20 66 6f 72 20 6f 6e 65 20 70 65 6f 70 6c 65 20 74 6f 20 
    64 69 73 73 6f 6c 76 65 20 74 68 65 20 70 6f 6c 69 74 69 63 61 6c 20 62 61 6e 
    64 73 20 77 68 69 63 68 20 68 61 76 65 20 63 6f 6e 6e 65 63 74 65 64 20 74 68 
    65 6d 20 77 69 74 68 20 61 6e 6f 74 68 65 72 2c 20 61 6e 64 20 74 6f 20 61 73 
    73 75 6d 65 20 61 6d 6f 6e 67 20 74 68 65 20 70 6f 77 65 72 73 20 6f 66 20 74 
    68 65 20 65 61 72 74 68 2c 20 74 68 65 20 73 65 70 61 72 61 74 65 20 61 6e 64 
    20 65 71 75 61 6c 20 73 74 61 74 69 6f 6e 20 74 6f 20 77 68 69 63 68 20 74 68 
    65 20 4c 61 77 73 20 6f 66 20 4e 61 74 75 72 65 20 61 6e 64 20 6f 66 20 4e 61 
    74 75 72 65 27 73 20 47 6f 64 20 65 6e 74 69 74 6c 65 20 74 68 65 6d 2c 20 61 
    20 64 65 63 65 6e 74 20 72 65 73 70 65 63 74 20 74 6f 20 74 68 65 20 6f 70 69 
    6e 69 6f 6e 73 20 6f 66 20 6d 61 6e 6b 69 6e 64 20 72 65 71 75 69 72 65 73 20 
    74 68 61 74 20 74 68 65 79 20 73 68 6f 75 6c 64 20 64 65 63 6c 61 72 65 20 74 
    68 65 20 63 61 75 73 65 73 20 77 68 69 63 68 20 69 6d 70 65 6c 20 74 68 65 6d 
    20 74 6f 20 74 68 65 20 73 65 70 61 72 61 74 69 6f 6e 2e 10 d2 85 d8 cc 04 1a 
    1b 09 42 cf 66 d5 e7 72 43 40 11 27 c2 86 a7 57 42 53 40 19 00 00 00 00 00 40 
    8f 40

    Formats~
    P2PKH :
    [0] 76 a9 14 4f 4d 42 55 44 53 01 fd bd 01 0a 96 03 57 68 65 6e 20 69 6e 88 ac
    [1] 76 a9 14 20 74 68 65 20 43 6f 75 72 73 65 20 6f 66 20 68 75 6d 61 6e 88 ac
    [2] 76 a9 14 20 65 76 65 6e 74 73 2c 20 69 74 20 62 65 63 6f 6d 65 73 20 88 ac
    [3] 76 a9 14 6e 65 63 65 73 73 61 72 79 20 66 6f 72 20 6f 6e 65 20 70 65 88 ac
    [4] 76 a9 14 6f 70 6c 65 20 74 6f 20 64 69 73 73 6f 6c 76 65 20 74 68 65 88 ac
    [5] 76 a9 14 20 70 6f 6c 69 74 69 63 61 6c 20 62 61 6e 64 73 20 77 68 69 88 ac
    [6] 76 a9 14 63 68 20 68 61 76 65 20 63 6f 6e 6e 65 63 74 65 64 20 74 68 88 ac
    [7] 76 a9 14 65 6d 20 77 69 74 68 20 61 6e 6f 74 68 65 72 2c 20 61 6e 64 88 ac
    [8] 76 a9 14 20 74 6f 20 61 73 73 75 6d 65 20 61 6d 6f 6e 67 20 74 68 65 88 ac
    [9] 76 a9 14 20 70 6f 77 65 72 73 20 6f 66 20 74 68 65 20 65 61 72 74 68 88 ac
    [10] 76 a9 14 2c 20 74 68 65 20 73 65 70 61 72 61 74 65 20 61 6e 64 20 65 88 ac
    [11] 76 a9 14 71 75 61 6c 20 73 74 61 74 69 6f 6e 20 74 6f 20 77 68 69 63 88 ac
    [12] 76 a9 14 68 20 74 68 65 20 4c 61 77 73 20 6f 66 20 4e 61 74 75 72 65 88 ac
    [13] 76 a9 14 20 61 6e 64 20 6f 66 20 4e 61 74 75 72 65 27 73 20 47 6f 64 88 ac
    [14] 76 a9 14 20 65 6e 74 69 74 6c 65 20 74 68 65 6d 2c 20 61 20 64 65 63 88 ac
    [15] 76 a9 14 65 6e 74 20 72 65 73 70 65 63 74 20 74 6f 20 74 68 65 20 6f 88 ac
    [16] 76 a9 14 70 69 6e 69 6f 6e 73 20 6f 66 20 6d 61 6e 6b 69 6e 64 20 72 88 ac
    [17] 76 a9 14 65 71 75 69 72 65 73 20 74 68 61 74 20 74 68 65 79 20 73 68 88 ac
    [18] 76 a9 14 6f 75 6c 64 20 64 65 63 6c 61 72 65 20 74 68 65 20 63 61 75 88 ac
    [19] 76 a9 14 73 65 73 20 77 68 69 63 68 20 69 6d 70 65 6c 20 74 68 65 6d 88 ac
    [20] 76 a9 14 20 74 6f 20 74 68 65 20 73 65 70 61 72 61 74 69 6f 6e 2e 10 88 ac
    [21] 76 a9 14 d2 85 d8 cc 04 1a 1b 09 42 cf 66 d5 e7 72 43 40 11 27 c2 86 88 ac
    [22] 76 a9 14 a7 57 42 53 40 19 00 00 00 00 00 40 8f 40 00 00 00 00 00 00 88 ac

    Null Data :
    FAIL


### Odd Bulletin 1
    Case title : OB1
    Case description :
        (1) valid bulletin
        (2) smallest bulletin possible
        (3) no location datum


### Odd Bulletin 2
    Case title : OB2
    Case description :
        (1) valid bulletin
        (2) largest bulletin possible (exactly 75KB)
        (3) includes location datum


### Odd Bulletin 3
    Case title : OB3
    Case description :
        (1) valid bulletin
        (2) less than 80 bytes in total length
        (3) includes funky characters in message
        (4) includes location datum


### Odd Bulletin 4
    Case title : OB4
    Case description :
        (1) valid bulletin
        (2) greater than 80 bytes in total length
        (3) includes funky characters in message
        (4) includes location datum


### Odd Bulletin 5
    Case title : OB5
    Case description :
        (1) valid bulletin
        (2) funky, small time
        (3) funky, small location datum


### Odd Bulletin 6
    Case title : OB6
    Case description :
        (1) valid bulletin
        (2) funky, large time
        (3) funky, large location datum
### Standard Endorsement 1
    Case title : SE1
    Case description :
        (1) valid endorsement
        (2) less than 80 bytes in total length


Invalid Records
---------------
special characters
missing data 
header type does not correspond to record type
header length does not correspond to record len
out of order enodings
greater than 75 kilobytes
impossible endorsements?
