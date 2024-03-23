We have seen operation for integers on elliptic curve, what amazing is that we can change those integers into finite field member without any problem. Remember that
field member with operation "+" and "." are normal arithmetic operation but with its result do the modulur operation. It turns out ellitpic curve point addition can
still hold even has its result base on modulur.

For example we can verify that finite fileld with order 103, we take two members 17 and 64 from this field and group them as a point coordinate (17, 64), this point
is on the curve y^2 = x^3+7 with modulur 103:

y^2 = 64^2 % 103 = 79, x^3+7 = (17^3+7) % 103 = 79

Now let's rewrite our Point struct with its components changed from big.Int to FieldElement, the changed code just like following:
```g
package elliptic_curve

import (
	"fmt"
	"math/big"
)

type OP_TYPE int

const (
	ADD OP_TYPE = iota
	SUB
	MUL
	DIV
	EXP
)

type Point struct {
	a *FieldElement
	b *FieldElement
	x *FieldElement
	y *FieldElement
}

func OpOnBig(x *FieldElement, y *FieldElement, scalar *big.Int, opType OP_TYPE) *FieldElement {
	switch opType {
	case ADD:
		return x.Add(y)
	case SUB:
		return x.Subtract(y)
	case MUL:
		/*
			Multiply in two cases, one is two field member multiply,
			the other is field member multiply with scalar
		*/
		if y != nil {
			return x.Multiply(y)
		}
		if scalar != nil {
			return x.ScalarMul(scalar)
		}
		panic("error in multiply")
	case DIV:
		return x.Divide(y)
	case EXP:
		if scalar == nil {
			panic("scalar should not be nil for EXP op")
		}
		return x.Power(scalar)
	}

	panic("should not come to here")
}

func NewEllipticCurvePoint(x *FieldElement, y *FieldElement, a *FieldElement, b *FieldElement) *Point {
	if x == nil && y == nil {
		return &Point{
			x: x,
			y: y,
			a: a,
			b: b,
		}
	}

	left := OpOnBig(y, nil, big.NewInt(int64(2)), EXP)
	x3 := OpOnBig(x, nil, big.NewInt(int64(3)), EXP)
	ax := OpOnBig(a, x, nil, MUL)
	right := OpOnBig(OpOnBig(x3, ax, nil, ADD), b, nil, ADD)
	if left.EqualTo(right) != true {
		err := fmt.Sprintf("Point(%v, %v) is not on the curve with a:%v, b:%v\n", x, y, a, b)
		panic(err)
	}

	return &Point{
		x: x,
		y: y,
		a: a,
		b: b,
	}

}

func (p *Point) Add(other *Point) *Point {
	//check two points are on the same curve
	if p.a.EqualTo(other.a) != true || p.b.EqualTo(other.b) != true {
		panic("given two points are not on the same curve")
	}

	if p.x == nil {
		return other
	}

	if other.x == nil {
		return p
	}

	//points are on the vertical A(x,y) B(x,-y),
	zero := NewFieldElement(p.x.order, big.NewInt(int64(0)))
	if p.x.EqualTo(other.x) == true &&
		OpOnBig(p.y, other.y, nil, ADD).EqualTo(zero) == true {
		return &Point{
			x: nil,
			y: nil,
			a: p.a,
			b: p.b,
		}
	}

	//find slope of line AB
	//x1 -> p.x, y1->p.y, x2 ->other.x, y2->other.y
	var numerator *FieldElement
	var denominator *FieldElement
	if p.x.EqualTo(other.x) == true && p.y.EqualTo(other.y) == true {
		//slope = (3*x^2+a)/2y
		xSqrt := OpOnBig(p.x, nil, big.NewInt(int64(2)), EXP)
		threeXSqrt := OpOnBig(xSqrt, nil, big.NewInt(int64(3)), MUL)
		numerator = OpOnBig(threeXSqrt, p.a, nil, ADD)
		//denominator: 2y
		denominator = OpOnBig(p.y, nil, big.NewInt(int64(2)), MUL)
	} else {
		numerator = OpOnBig(other.y, p.y, nil, SUB)   //(y2-y1)
		denominator = OpOnBig(other.x, p.x, nil, SUB) //(x2-x1)
	}

	//s=(y2-y1) / (x2-x1)
	slope := OpOnBig(numerator, denominator, nil, DIV)
	//s^2
	slopeSqrt := OpOnBig(slope, nil, big.NewInt(int64(2)), EXP)
	//x3 = s^2 - x1 - x2
	x3 := OpOnBig(OpOnBig(slopeSqrt, p.x, nil, SUB), other.x, nil, SUB)
	// x3-x1
	x3Minusx1 := OpOnBig(x3, p.x, nil, SUB)
	//y3 = s(x3-x1)+y1
	y3 := OpOnBig(OpOnBig(slope, x3Minusx1, nil, MUL), p.y, nil, ADD)
	//-y3
	minusY3 := OpOnBig(y3, nil, big.NewInt(int64(-1)), MUL)

	return &Point{
		x: x3,
		y: minusY3,
		a: p.a,
		b: p.b,
	}
}

func (p *Point) String() string {
	xString := "nil"
	yString := "nil"
	if p.x != nil {
		xString = p.x.String()
	}
	if p.y != nil {
		yString = p.y.String()
	}
	return fmt.Sprintf("(x:%s\n, y:%s\n, a:%s\n, b:%s\n)", xString, yString,
		p.a.String(), p.b.String())
}

func (p *Point) Equal(other *Point) bool {
	if p.a.EqualTo(other.a) == true && p.b.EqualTo(other.b) == true &&
		p.x.EqualTo(other.x) == true &&
		p.y.EqualTo(other.y) == true {
		return true
	}

	return false
}

func (p *Point) NotEqual(other *Point) bool {
	if p.a.EqualTo(other.a) != true || p.b.EqualTo(other.b) != true ||
		p.x.EqualTo(other.x) != true ||
		p.y.EqualTo(other.y) != true {
		return true
	}

	return false
}

```
We change the code accordingly and no new things there, let's test the changed code as following:
```g
package main

import (
	ecc "elliptic_curve"
	"fmt"
	"math/big"
)

func main() {
	x := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(192)))
	y := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(105)))
	a := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(0)))
	b := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(7)))
	p1 := ecc.NewEllipticCurvePoint(x, y, a, b)

	fmt.Printf("elliptic curve point over finite field is %s\n", p1)

	yNeg := y.Negate()
	p2 := ecc.NewEllipticCurvePoint(x, yNeg, a, b)
	res := p1.Add(p2)
	fmt.Printf("addition of points on vertical line over finit field is %s\n", res)

	x2 := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(17)))
	y2 := ecc.NewFieldElement(big.NewInt(int64(223)), big.NewInt(int64(56)))
	p3 := ecc.NewEllipticCurvePoint(x2, y2, a, b)
	res = p1.Add(p3)
	fmt.Printf("p1 + p2 over field order 223 is %s\n", res)

	res = p1.Add(p1)
	fmt.Printf("p1 + p1 over field order 223 is %s\n", res)
}
```
run the code and we get the result:
```g
elliptic curve point over finite field is (x:FieldElement{order: 223, num: 192}

, y:FieldElement{order: 223, num: 105}

, a:FieldElement{order: 223, num: 0}

, b:FieldElement{order: 223, num: 7}

)
addition of points on vertical line over finit field is (x:nil
, y:nil
, a:FieldElement{order: 223, num: 0}

, b:FieldElement{order: 223, num: 7}

)
p1 + p2 over field order 223 is (x:FieldElement{order: 223, num: 170}

, y:FieldElement{order: 223, num: 142}

, a:FieldElement{order: 223, num: 0}

, b:FieldElement{order: 223, num: 7}

)
p1 + p1 over field order 223 is (x:FieldElement{order: 223, num: 49}

, y:FieldElement{order: 223, num: 71}

, a:FieldElement{order: 223, num: 0}

, b:FieldElement{order: 223, num: 7}

)
```
I can make you at rest that I have checked those result with a pen and paper, trust me!
