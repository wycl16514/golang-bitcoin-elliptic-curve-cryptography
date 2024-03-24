Let's see how bitcoin rely on elliptic curve for cryptography especially generate public key for a wallet and private key for signing transaction and verify the
legality of a transaction, these envolve following steps:

1, set a = 0 and b = 7 to fix its elliptic curve as y^2 = x^3 + 7
2, specify a huge prime p for the finite field order, p = 2^256 - 2^32 - 977
3, select a point from the curve as a generator point G, the x coordinate of the point is 0x79be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798,
and the y coordinate of the point is 0x483ada7726a3c4655da4fbfc0e1108a8fd17b448a68554199c47d08ffb10d4b8
4, generate the group by multiply G until it turns into identity, the value of n such that n * G is identity is:
0xfffffffffffffffffffffffffffffffebaaedce6af48a03bbfd25e8cd0364141

Notice that p is extremely close to 2^256, which means the x and y coordinate of G can be represented by 256 bits integer, and you can see 64bits integer are far
from enough to be useful in bitcoin and that's why we resort to big.Int for help. We can select a number k in the range of (1,n) as the private key, and compute
k*G as the public key, in later section we will see how to generate a wallet address by using the public key.

First let's use code to check the point G is really on the curve of y^2 = x^3 + 7:
```g
func main() {
	var op big.Int
	twoExp256 := op.Exp(big.NewInt(int64(2)), big.NewInt(int64(256)), nil)
	var op1 big.Int
	twoExp32 := op1.Exp(big.NewInt(int64(2)), big.NewInt(int64(32)), nil)
	var op2 big.Int
	p := op2.Sub(twoExp256, twoExp32)
	var op3 big.Int
	pp := op3.Sub(p, big.NewInt(int64(977)))
	fmt.Printf("pp is %s\n", pp)
	Gx := new(big.Int)
	Gx.SetString("79be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798", 16)
	fmt.Printf("Gx :%s\n", Gx)
	Gy := new(big.Int)
	Gy.SetString("483ada7726a3c4655da4fbfc0e1108a8fd17b448a68554199c47d08ffb10d4b8", 16)
	fmt.Printf("Gy: %s\n", Gy)
	x1 := ecc.NewFieldElement(pp, Gx)
	y1 := ecc.NewFieldElement(pp, Gy)
	a := ecc.NewFieldElement(pp, big.NewInt(int64(0)))
	b := ecc.NewFieldElement(pp, big.NewInt(int64(7)))
	G := ecc.NewEllipticPoint(x1, y1, a, b)
	fmt.Printf("G on bitcoin elliptic point is   %s\n", G)

}
```
Running above code gives the result :
```g
pp is 115792089237316195423570985008687907853269984665640564039457584007908834671663
Gx :55066263022277343669578718895168534326250603453777594175500187360389116729240
Gy: 32670510020758816978083085130507043184471273380659243275938904335757337482424
G on bitcoin elliptic point is
 (x: FieldElement{order: 115792089237316195423570985008687907853269984665640564039457584007908834671663,
num: 55066263022277343669578718895168534326250603453777594175500187360389116729240},
y: FieldElement{order: 115792089237316195423570985008687907853269984665640564039457584007908834671663,
num: 32670510020758816978083085130507043184471273380659243275938904335757337482424},
a: FieldElement{order: 115792089237316195423570985008687907853269984665640564039457584007908834671663, num: 0},
b: FieldElement{order: 115792089237316195423570985008687907853269984665640564039457584007908834671663, num: 7})
```
We can see that's crazy huge number for human.
