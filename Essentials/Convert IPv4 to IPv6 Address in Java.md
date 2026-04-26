# Convert IPv4 to IPv6 Address in Java | Baeldung

## 1\. Overview

Internet Protocol version 6 or IPv6, defined in RFC 2460, is the latest generation of the [Internet Protocol](/cs/popular-network-protocols#internet-protocol-ip). It replaces the limited IPv4 address space. However, [IPv6](/cs/ipv4-vs-ipv6#ipv6) doesn’t directly interoperate with [IPv4](/cs/ipv4-datagram#ipv4-datagram). Because of this, devices using different protocols often face compatibility issues.

To solve this, modern systems use techniques like dual-stack, tunneling, and translation. Thus, these methods enable systems to support IPv6 while still working with IPv4. However, **they primarily work at the [Network Layer](/cs/osi-transport-vs-networking-layer#services-of-network-layer) (Layer 3)**.

In many cases, developers need a way to represent IPv4 addresses in an IPv6-compatible format. So, they write code to map data between the two formats.

In this tutorial, we see how to convert an IPv4 address to IPv6 using [Java](/get-started-with-java-series).

## 2\. What Does Converting Mean?

In reality, **there is no direct conversion from IPv4 to IPv6**. The two protocols differ in structure and design.

Instead, a common approach is to embed IPv4 inside IPv6. This format is called an IPv4-mapped IPv6 address.

In addition, we also have other representations. However, not all are standard today. For example, **IPv4-compatible IPv6 addresses are now deprecated**. In the same way, _NAT64_ and _6to4_ act as transition methods, not true conversions.

So, let’s look at how to implement the above techniques in Java.

## 3\. Java Implementation

In essence, Java provides built-in classes to work with IP addresses. The [_InetAddress_](/java-get-ip-url#1-using-the-inetaddress-class) class supports both IPv4 and IPv6.

### 3.1. Convert IPv4 to IPv6-Mapped Address

Let’s look at the case of a mapped address.

For that, we convert an IPv4 address to an IPv6 format using a standard format:

```
::ffff:w.x.y.z
```

In the above case, _w.x.y.z_ is the target IPv4 address:

```
String toIpv4MappedIpv6(String ipv4Address) throws UnknownHostException {
    validateIpv4(ipv4Address);
    return "::ffff:" + ipv4Address;
}
```

**This method simply adds the _::ffff:_ prefix**. As a result, we get a valid IPv4-mapped IPv6 address.

### 3.2. IPv4-Compatible IPv6 Address

Markedly, the IPv4-compatible IPv6 address format is no longer in use. Still, it helps to understand older systems.

**In hex notation, this type of address sets all the leading bits to zero**.

Let’s again take the IPv4 address _w.x.y.z_:

```
0000:0000:0000:0000:0000:0000:WWXX:YYZZ
```

This address can also be compressed by omitting all leading zeroes:

```
::w.x.y.z 
```

Furthermore, we see its implementation:

```
String toIpv4CompatibleIpv6(String ipv4Address) throws UnknownHostException {
    validateIpv4(ipv4Address);
    return "::" + ipv4Address;
}
```

The above method takes an IPv4 address and returns an IPv6 address.

As a result, an IPv4 address, _192.0.2.33_, returns IPv4-compatible IPv6 _::192.0.2.33._

### 3.3. NAT64 Implementation

NAT64-style translated addresses typically use a _64:ff9b::/96_ prefix. It’s then followed by the target IPv4 address.

For example, **an IP address of the form _w.x.y.z_ gets the NAT64 ID _64:ff9b::w.x.y.z_**.

The implementation code remains the same as the last ones:

```
String toNat64Ipv6(String ipv4Address) throws UnknownHostException {
    validateIpv4(ipv4Address);
    return "64:ff9b::" + ipv4Address;
}
```

Thus, if we have an IPv4 address of _192.0.2.33_, the resulting address should be _64:ff9b::192.0.2.33_.

### 3.4. _6to4_ Addresses (Tunneling)

_6to4_ is a transition method. **It wraps IPv6 packets inside IPv4 packets**. Moreover, for this purpose, Internet Assigned Numbers Authority ([IANA](https://www.iana.org/)) has assigned the _2002::/16_ block.

For example, the IPv4 address _192.0.2.4_ has a hexadecimal equivalent of _C000:0204_. **The _6to4_ prefix in this case becomes _2002:C000:0204::/48_**:

```
16 bits (the base) + 32 bits (the IPv4) = 48 bits
```

Thus, the prefix _2002::/16_ is followed by the _32_ bits of the public IPv4 address.

Let’s see its Java implementation:

```
String toSixToFourIpv6(String ipv4Address) throws UnknownHostException {
    byte[] bytes = validateAndGetBytes(ipv4Address);
    return String.format("2002:%02x%02x:%02x%02x::",
      bytes[0] & 0xff,
      bytes[1] & 0xff,
      bytes[2] & 0xff,
      bytes[3] & 0xff);
}
```

The above method converts each IPv4 byte into its hexadecimal representation.

Finally, the result gets appended to the _2002::/16_ prefix.

## 4\. JUnit Test

To verify the class methods, we can write a simple JUnit test:

```
class Ipv4ToIpv6ConverterUnitTest {
    private final Ipv4ToIpv6Converter converter = new Ipv4ToIpv6Converter();
    @Test
    void whenValidIpv4_thenReturnMappedIpv6() throws Exception {
        String result = converter.toIpv4MappedIpv6("192.168.1.1");
        assertEquals("::ffff:192.168.1.1", result);
    }
}
```

Thus, we can check if the output matches the expected value. In the same way, we can write tests for other methods.

## 5\. Use Cases

IPv4-mapped IPv6 addresses help both protocols work together in dual-stack systems.

Users can also use a tunneling mechanism, like _6to4_, if they don’t have dual-stack support.

Similarly, we have translators like NAT64. They also help in interoperability between an IPv4 device and an IPv6 device.

The general idea of the conversion is to enable the different protocols to work in sync.

## 6\. Conclusion

In this article, we discussed the conversion between IPv4 and IPv6 with Java. Since both IP protocols differ in structure and semantics, we can’t truly convert IPv4 into IPv6.

Because of this, **the above approaches don’t turn an IPv4 host into a native IPv6 host**. Instead, they provide a way for the systems to work together. Java simplifies this process through classes like InetAddress and byte handling.

As a result, applications can support both address formats efficiently.

The complete code for this article is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-5).