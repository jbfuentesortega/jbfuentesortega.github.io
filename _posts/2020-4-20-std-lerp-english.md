---
layout: post
title: 'std::lerp and why you might not want to use it'
date: 2020-04-20
tags:
    c++
	english
---
C++20 introduces `std::lerp`, a function that, given three numbers a, b, and t, calculates the linear interpolation between a and b at t. It includes overloads for all number type combinations in the language and does what it has to do. What's more, it does it very well. It does it so well that for all t between 0 and 1 the result is **always** correct.

**Part 1: `std::lerp` may be unnecessarily slow**

If we implement this function, we will most likely write something like this:

```cpp
float lerp(float a, float b, float t)
{
	return a + (b – a) * t;
}
```

This code seems correct. Well, is it? It is in all cases except when the intermediate value (b - a) is less than what a float can represent. In that case, the calculation is wrong. `std::lerp` instead calculates this case correctly and returns the correct result even in situations of unrepresentable intermediate values. Knowing this, one would think that `std::lerp` is perfect. Not only does it come ready-made, but it is also more correct than the version that we would write. So, where is the problem?

Well, the problem is simple. This guarantee has a cost. In order to ensure that it will not fall into a situation where one of the intermediate values ​​is not representable, `std::lerp` analyzes its parameters and decides which formula to use to calculate linear interpolation so that it always choose a correct one. This makes `std::lerp` significantly slower than our version. This can be seen very well in the assembly.

```
our_lerp(float, float, float):
        subss   xmm1, xmm0
        mulss   xmm1, xmm2
        addss   xmm0, xmm1
        ret




std_lerp(float, float, float):
        xorps   xmm3, xmm3
        ucomiss xmm3, xmm0
        jb      .LBB0_2
        ucomiss xmm1, xmm3
        jae     .LBB0_4
.LBB0_2:
        ucomiss xmm0, xmm3
        jb      .LBB0_5
        xorps   xmm3, xmm3
        ucomiss xmm3, xmm1
        jb      .LBB0_5
.LBB0_4:
        mulss   xmm1, xmm2
        movss   xmm3, dword ptr [rip + .LCPI0_0] # xmm3 = mem[0],zero,zero,zero
        subss   xmm3, xmm2
        mulss   xmm3, xmm0
        addss   xmm1, xmm3
        movaps  xmm0, xmm1
        ret
.LBB0_5:
        ucomiss xmm2, dword ptr [rip + .LCPI0_0]
        jne     .LBB0_6
        jp      .LBB0_6
        movaps  xmm0, xmm1
        ret
.LBB0_6:
        movaps  xmm3, xmm1
        subss   xmm3, xmm0
        mulss   xmm3, xmm2
        ucomiss xmm2, dword ptr [rip + .LCPI0_0]
        setbe   al
        ucomiss xmm1, xmm0
        addss   xmm3, xmm0
        seta    cl
        xor     cl, al
        jne     .LBB0_7
        minss   xmm3, xmm1
        movaps  xmm1, xmm3
        movaps  xmm0, xmm1
        ret
.LBB0_7:
        maxss   xmm3, xmm1
        movaps  xmm1, xmm3
        movaps  xmm0, xmm1
        ret
```

While our lerp only does a subtraction, a multiplication and a sum, we can see how `std::lerp` has a series of conditional jumps and four branches that it can end in. This increase in cost should make us wonder how useful this additional guarantee that it gives us really is, and base our decision on this.

One of the most common situations in which linear interpolation is used is in skeleton and particle system animations. In these, lerp is the most common operation, and is calculated potentially several thousand times per frame. Also, it is generally operated with values ​​known in advance, decided by hand by a designer, an animator or a special effects artist, so it is known that these values ​​will never be so large that they are not representable by the type of data being used. In these cases, the decision to use std :: lerp is wrong, as an additional cost is being introduced into the code without this reporting any benefits in return.

I'm not saying that `std::lerp` is wrong or that you shouldn't use it. If so, it is misunderstood. Maybe incorrectly named. The feature that defines `std::lerp` is not that it calculates linear interpolation, but that it does so correctly in all situations, and so it should be used only when there is a possibility that there is a case in which the lerp implementation shown up break. In cases where it is known in advance that this type of error will not occur, the decision of using `std::lerp` is wrong.

**Part 2: `std::lerp` is incorrectly abstracted**

Linear interpolation is an operation defined on a set V that defines the sum and the dot product. This set is obviously a vector space. However, `std::lerp` is defined only for numbers. This prevents us from linearly interpolating vectors, polynomials, quaternions or RGB colors, all of them very frequent candidates for linear interpolation in real applications. A correct definition of lerp in C++, assuming the VectorSpace and Real concepts, would be

```cpp
template <VectorSpace V, Real R>
V lerp(V a, V b, R t)
{
	return a + (b – a) * t;
}
```

I suspect the current implementation is due to having to carry out these special case checks, which would not be possible on an arbitrary vector space.