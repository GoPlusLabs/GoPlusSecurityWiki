---
coverY: 0
---

# Zero-knowledge Proof

## Definition of ZKP Security

In a ZKP implementation, three core principles of ZKP should be achieved to fulfil ZKP Security:

### Soundness

* All invalid proofs must always be rejected
* Valid proofs should not be faked, modified or replayed

#### Negative Examples

* Constrains are compromised
* Proving keys are generated insecurely or sealed in an unsafe way

### Zero-knowledge

* Witness information shouldn't leak in any other place, eg. in a proof

#### Negative Examples

* Private variable is published
* Metedata attack

### Completeness

* All valid proofs must always be accepted
* All circuits or programmes should be handled correctly

#### Negative Examples

* R1CS incorrect generation&#x20;
* gadget i/o value mismatch causing gadget combination fails

### Other negatives

* Data leakage through side channels or encodings
* Any unsafe state(code execution, DoS)
* Trusted setup hack
* build and release integrity
* software dependencies/libraries





