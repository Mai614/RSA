# RSA
Implement RSA algorithm
import random


def gcd(a, b):
    while b != 0:
        a, b = b, a % b
    return a


def modInverse(a, m):
    m0 = m
    y = 0
    x = 1

    if (m == 1):
        return 0

    while (a > 1):
        # q is quotient
        q = a // m

        t = m

        # m is remainder now, process
        # same as Euclid's algo
        m = a % m
        a = t
        t = y

        # Update x and y
        y = x - q * y
        x = t

    # Make x positive
    if (x < 0):
        x = x + m0
    return x


def s_a_m(base, exponent, modulo):
    shift = 511
    result = base
    while ((exponent >> shift) & 1) == 0:
        shift = shift - 1
    shift = shift - 1
    while shift >= 0:
        result = (result * result)
        if modulo != -1:
            result = result % modulo
        if ((exponent >> shift) & 1) == 1:
            result = (result * base)
            if modulo != -1:
                result = result % modulo
        shift = shift - 1
    return result


def CRT(y, d, p, q):
    n = p * q
    yp = y % p
    yq = y % q
    dp = d % (p - 1)
    dq = d % (q - 1)
    xp = s_a_m(yp, dp, p)
    xq = s_a_m(yq, dq, q)
    cp = modInverse(q, p)
    cq = modInverse(p, q)
    x = (q * cp * xp + p * cq * xq) % n
    return x


def miller_rabin(p, s):
    if p == 2 or p == 3:
        return True

    if p < 2 or p % 2 == 0:
        return False

    u, r = 0, p - 1
    while r % 2 == 0:
        u += 1
        r //= 2
    for _ in range(s):
        a = random.randrange(2, p - 2)
        z = s_a_m(a, r, p)
        if z == 1 or z == p - 1:
            continue
        for _ in range(u - 1):
            z = s_a_m(z, 2, p)
            if z == 1:
                return False
        if z != p-1:
            return False

    return True


def generate_large_primes():
    p = random.randrange(s_a_m(2, 255, -1), s_a_m(2, 256, -1))
    while miller_rabin(p, 200) == False:
        p = random.randrange(s_a_m(2, 255, -1), s_a_m(2, 256, -1))

    q = random.randrange(s_a_m(2, 255, -1), s_a_m(2, 256, -1))
    while miller_rabin(q, 200) == False:
        q = random.randrange(s_a_m(2, 255, -1), s_a_m(2, 256, -1))

    return p, q


def generate_public_key(phi):
    pub_key = 2
    while (gcd(phi, pub_key) != 1):
        pub_key = pub_key + 1
    return pub_key

def generate_privet_key(e, phi):
    prv_key =modInverse(e, phi)
    return prv_key

def encrypt(x, e, n):
    y = s_a_m(x, e, n)
    return y


def decrypt(y, d, n):
    x = CRT(y, d, p, q)
    return x


p, q = generate_large_primes()
phi = (p - 1) * (q - 1)
n = p * q
e = generate_public_key(phi)
d = generate_privet_key(e, phi)
x = int(input('Enter the message to be encrypted :'))
y = encrypt(x, e, n)
print('The encrypted message is :')
print(y)
x = decrypt(y, d, n)
print('The decrypted message is :')
print(x)
