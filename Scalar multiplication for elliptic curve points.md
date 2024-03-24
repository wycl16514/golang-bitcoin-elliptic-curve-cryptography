There is a very important oerpation base on addition for elliptic curve points that is multiplication. The multiply of curve points will have important impact for
bitcoin application. Multiply with scalar is equavalent for doing the same times of addition. Elliptic curve cryptography is base on scalar multiplication on points.

There is a nice property for scalar multiplication. Select a point from the curve and keep doing multiplication for a certain times and you will get an identity point:
{G, 2G, ... , nG} , and nG is an identity point which we can define nG = 0, and there is a mathmatic name for the set {G, 2G, ..., nG} that is group.

Given a point G and a scalar k, we can do the multiply very easily k * G = Q, but if given G and Q, it is impossible to comute the k from G and Q,
this property is why we can do cryptography base on elliptic curve.

Let's see how we code the scalar multiply for elliptic point, intutively we can do scalar multiply by doing the same times of addition, but there is a problem for this
approch, what if the scalar is overly huge such as one trillion? repeating on trillion times of addition will make our computer stuck even if have the latest mac worth
of several thousand dollors.

We need to resort to a trick called binary expansion, for the scalar k, we turn it into its binary form, for eample for number 13, its binary form is:
2^3 + 2^2 + 2^0, for 13*G is equivalent to 2^3*G + 2^2*G + G and is equivalent to (G<<3) + (G<<2) + (G<<0) , you can see we can turn the multiplication into addition
with t times, and t is how many 1 in the binary form of k, this greatly reduce the time we need to do multiply. For example if k is on trillion, then it has most 40 bits
which means it has most 40 1s in its binary form which means we only need to do no more than 40 times of addition compare with on trillion times of addition before!

Let's see how we can put that trick into code:
```g
func (p *Point) ScalarMul(scalar *big.Int) *Point {
	if scalar == nil {
		panic("scalar mul error ofr nil scalar")
	}
	/*
		turn scalar into binary string form, for example 13 will turn into "1101"
	*/
	binaryForm := fmt.Sprintf("%b", scalar)
	current := p
	result := NewEllipticPoint(nil, nil, p.a, p.b)
	for i := len(binaryForm) - 1; i >= 0; i-- {
		if binaryForm[i] == '1' {
			result = result.Add(current)
		}
		//add itself is the same as left shift 1 bit
		current = current.Add(current)
	}

	return result
}
```
Let's run test the code and get the result:
```g
package main

import (
	ecc "elliptic_curve"
	"fmt"
	"math/big"
)

func main() {
	/*
		2*(192,105), 2*(143,98), 2*(47,71)
	*/
	x1 := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(192)))
	y1 := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(105)))
	a := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(0)))
	b := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(7)))
	p1 := ecc.NewEllipticPoint(x1, y1, a, b)
	fmt.Printf("2*(192,105) is  %s\n", p1.ScalarMul(big.NewInt(int64(2))))

	x2 := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(143)))
	y2 := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(98)))
	p2 := ecc.NewEllipticPoint(x2, y2, a, b)

	fmt.Printf("2*(143,98) is %s\n", p2.ScalarMul(big.NewInt(int64(2))))

	x3 := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(47)))
	y3 := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(71)))
	p3 := ecc.NewEllipticPoint(x3, y3, a, b)
	fmt.Printf("2*(47,71) is %s\n", p3.ScalarMul(big.NewInt(int64(2))))
	/*
		    practise:
			4*(47, 71), 8*(47,71), 21*(47,71)
	*/
	x4 := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(47)))
	y4 := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(71)))
	p4 := ecc.NewEllipticPoint(x4, y4, a, b)
	fmt.Printf("4*(47,71) is %s\n", p4.ScalarMul(big.NewInt(int64(4))))
	fmt.Printf("8*(47,71) is %s\n", p4.ScalarMul(big.NewInt(int64(8))))
	fmt.Printf("21*(47,71) is %s\n", p4.ScalarMul(big.NewInt(int64(21))))
}
```
The result of running above code is :
```g
2*(192,105) is  (x: FieldElement{order: 223, num: 49}, y: FieldElement{order: 223, num: 71}, a: FieldElement{order: 223, num: 0}, b: FieldElement{order: 223, num: 7})

2*(143,98) is (x: FieldElement{order: 223, num: 64}, y: FieldElement{order: 223, num: 168}, a: FieldElement{order: 223, num: 0}, b: FieldElement{order: 223, num: 7})

2*(47,71) is (x: FieldElement{order: 223, num: 36}, y: FieldElement{order: 223, num: 111}, a: FieldElement{order: 223, num: 0}, b: FieldElement{order: 223, num: 7})

4*(47,71) is (x: FieldElement{order: 223, num: 194}, y: FieldElement{order: 223, num: 51}, a: FieldElement{order: 223, num: 0}, b: FieldElement{order: 223, num: 7})

8*(47,71) is (x: FieldElement{order: 223, num: 116}, y: FieldElement{order: 223, num: 55}, a: FieldElement{order: 223, num: 0}, b: FieldElement{order: 223, num: 7})

21*(47,71) is (x: nil, y: nil, a: FieldElement{order: 223, num: 0}, b: FieldElement{order: 223, num: 7})

```

