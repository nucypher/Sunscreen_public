# Sunscreen 
:sunny: **Secure, Private, Flexible Smart Contracts** :sunny:

This repository contains work on privacy-preserving smart contracts (PPSCs). We've included the 2 documents that cover [zero-knowledge proof systems](https://en.wikipedia.org/wiki/Zero-knowledge_proof), [Bulletproofs](https://eprint.iacr.org/2017/1066.pdf), and a rough outline on our vision for secure, private, flexible smart contracts. We wrote these in what we hope is an accessible way.  

Previous talks on this subject were given at [IEEE S&P 2020](https://www.ieee-security.org/TC/SP2020/program-shorttalks.html) (as a short talk; "The Marriage of Fully Homomorphic Encryption and Blockchain"), [ZKSummit](https://www.zeroknowledge.fm/) ("The Future of PPSCs"), and  [Devcon](https://devcon.org/) ("The Future of PPSCs").

We include a chart below for understanding what goes into our definitions of efficiency, security, privacy, and flexibility when discussing PPSCs. We call our project "Sunscreen" as our work will aim to satisfy 3 out of 4 of the goals defined&mdash;namely security, privacy, and flexibility&mdash;while sacrificing some degree of efficiency.


| Efficiency                | Security Assumptions                   | Privacy                  | Flexibility                    |
| ------------------------- | :--------------------------: | :----------------------: | :----------------------------: |
|  Communication complexity?  |  Based on cryptography?     |       Confidential?       | Easily adaptable to new environments(i.e. universal reference string)? |
| Reference string size?       |     Based on hardware?      |             Anonymous?       | Supports arbitrary logic/computation?  |
| Setup time?                  |      Provable security? |     Based on cryptography?    |  Supports "stateful" computation?           |
| Time to generate transactions?   |   Non-standard assumptions?   |    Using tumblers or mixers?  |             |
| Time to verify transactions?     |     Post-quantum?  |  Using stealth addresses? |      |
| Physical resources required?    |    Trusted setup?      |   Function privacy?        |                  |
| Potential for scalability?          |                       |            |                |

## Background
We provide a brief overview of the contents of the 2 documents below. 

**Disclaimer: These documents were originally written for internal usage and may no longer reflect up to date information or ideas.**

### April 2019: "[Zero-Knowledge Proofs for PPSCs and Transactions](/zk%20thoughts.pdf)"

We look at some efficient zero-knowledge proof protocols (i.e. [SNARKs](https://z.cash/technology/zksnarks/), [STARKs](https://eprint.iacr.org/2018/046.pdf), [Bulletproofs](https://eprint.iacr.org/2017/1066.pdf)) to explore which might be the most promising for a private transaction/smart contract scheme. We also evaluate some recent projects in the space including [Zether](https://eprint.iacr.org/2019/191.pdf), [Hawk](https://eprint.iacr.org/2015/675.pdf), [Quisquis](https://eprint.iacr.org/2018/990.pdf), [Aztec](https://github.com/AztecProtocol/AZTEC/blob/develop/AZTEC.pdf), and [Zexe](https://eprint.iacr.org/2018/962.pdf).

### October 2019: "[The Future of Privacy-Preserving Smart Contracts](/Future_of_PPSCs.pdf)"

We outline our design goals for a PPSC scheme and our core beliefs about what a "successful" scheme might look like for us. We provide a brief look into the building blocks we might use for such a scheme and the associated challenges around combining efficient ZKPs with FHE.

The appendix of this document was originally part of a separate internal document titled "Using Bulletproofs" from July 2019. In it, we consider two lines of work based on Bulletproofs. The first builds on [Zether](https://eprint.iacr.org/2019/191.pdf) whereas the second argues for the creation of a new PPSC scheme using FHE based on [Short Discrete Log Proofs for FHE & Ring-LWE Ciphertexts](https://eprint.iacr.org/2019/057.pdf).


## Implementation Work
This section includes protyping and benchmarking work for Short Discrete Log Proofs, and BGV. Current prototyping is being done in Julia by [Bogdan](https://github.com/fjarri) for ease of iteration. However, we expect final libraries to be done in C/Rust. 

### [Short Discrete Log Proofs](https://eprint.iacr.org/2019/057.pdf)
A PoC of [Short Discrete Log Proofs](https://eprint.iacr.org/2019/057.pdf) can be found [here](https://github.com/nucypher/LogProof.jl). We provide charts below on the performance of the verifiable encryption/decryption scheme of Section 1.5 of the paper. We stress that our prototype is *not secure* and should *not* be used in production.

**Proof Performance**

Recall that the proofs use discrete logs and thus the performance is dependent on the curve chosen. 

[PoC](https://github.com/nucypher/LogProof.jl); Curve25519; Intel i7 @ 2.6GHz
|     Curve25519                      | 1 thread                  | 6 threads                    |
| -------------------------  | :----------------------: | :----------------------------: |
| Prover time                |     34.6s                |     8.2s                      |
| Verifier time                |     23.7s                |     5.2s                      |
| Initial proof generation               |     2.15s               |     434ms            |   

[PoC](https://github.com/nucypher/LogProof.jl); secp256k1; Intel i7 @ 2.6GHz
|     Secp256k1                      | 1 thread                  | 6 threads                    |
| -------------------------  | :----------------------: | :----------------------------: |
| Prover time                |     70s                |     14.9s                      |
| Verifier time                |     47s                |     9.7s                      |
| Initial proof generation               |     16s               |     3.23s            |   


**Encryption Scheme Performance**

The encryption scheme is solely based on lattices (specifically ring-lwe) and thus not dependent on the curve choice.

|   Ring-LWE Encryption Scheme      |       Timing (mean/median)            |
| -------------------------  |  :----------------------------: |
| Encrypt               |      1.294ms; 1.235ms                    |
| Decrypt             |    591.261microsec; 577.905microsec                    |


### [BGV Scheme](https://eprint.iacr.org/2011/277.pdf) [WIP]

**Benchmarking from [HElib](https://github.com/shaih/HElib)**


## Challenges

**FHE Requisite Knowledge**

Even some of the [nicest FHE libraries currently available](https://github.com/microsoft/SEAL) are challenging to work with&mdash;requiring deep expertise in the scheme to achieve optimal performance. While both proponents and critics of FHE argue that FHE isn't being used due to *efficiency* reasons, we disagree as FHE *can* provide acceptable runtimes for certain use cases. Rather, we believe what's preventing FHE from gaining more traction is the *level of expertise* required to currently use the schemes/libraries.

**Lack of Consistent Benchmarking**

Additionally, there's a dearth of work in benchmarking some of the recent ZKPs and FHE schemes. It's difficult to incorporate (or even know *if* we should incorporate) these works when we don't know how they perform in practice. The efficiency tables in our PATS repo (such as [this](https://github.com/ravital/pats/blob/master/zether.md), and [this](https://github.com/ravital/pats/blob/master/aztec.md)) have many blanks as the authors have failed to provide important estimates in their papers. Many of the PPSC works also use vastly different machines&mdash;from an [Intel i7-6820HQ throttled to 2.0 Ghz with <100MB of memory and a single thread](https://github.com/ravital/pats/blob/master/zether.md) to an [Intel Xeon 6138 CPU at 3.0 Ghz with 252GB of RAM and 12 threads](https://github.com/ravital/pats/blob/master/zexe.md)&mdash;further exacerbating the problem.

## Future Directions

Some directions of research we'd like to pursue from here include:
- Tools for making FHE easier to work with
- Benchmarking ZKPs (possibly an entire project in and of itself!)
- Benchmarking FHE schemes (i.e. FV, BGV)
- Development of a user-friendly BGV library

## Comments or Concerns
For any comments or concerns, feel free to open issues or ask questions on our [Discord channel](https://discord.gg/7rmXa3S). Otherwise, you can contact Ravital (ravital@nucypher.com) with general questions or for questions w.r.t. the research/documents. Questions about the specifics of the prototype/implementation can be directed to Bogdan (bogdan@nucypher.com).

## Background Reading
Resources that are beginner friendly, we mark with :green_apple:. More technically demanding resources we mark with :apple:.

We suggest the following resources:
- [ZKProof Community Reference](https://docs.zkproof.org/assets/docs/reference-v0.2.pdf), a fairly comprehensive document covering zero-knowledge proofs from the academic initiative [ZKProof](https://zkproof.org/) :green_apple:
- [Bulletproofs notes](https://doc-internal.dalek.rs/bulletproofs/notes/index.html), notes on how Bulletproofs work :apple:
- [A Decade of Lattice Cryptography](https://web.eecs.umich.edu/~cpeikert/pubs/lattice-survey.pdf), a comprehensive document explaining the basics of lattices, assumptions used in lattice cryptography, and corresponding schemes :apple:
- [Fundamentals of Fully Homomorphic Encryption - A Survey](https://pdfs.semanticscholar.org/e247/ae732c50b6c04b2aa413c4caa0ca77ed4751.pdf), an accessible but long introduction to FHE :green_apple:
- [Computing Arbitrary Functions of Encrypted Data](https://crypto.stanford.edu/craig/easy-fhe.pdf), a brief but well-written paper from Craig Gentry covering the basics of how FHE works :green_apple:
- [A brief survey of Fully Homomorphic Encryption](https://blog.quarkslab.com/a-brief-survey-of-fully-homomorphic-encryption-computing-on-encrypted-data.html), a short and easy-to-read post covering the basic aspects of FHE and some of the challenges associated with it :green_apple:
- [Microsoft's SEAL library](https://github.com/microsoft/SEAL), particularly the [example sections](https://github.com/microsoft/SEAL/tree/master/native/examples) to see how a FHE scheme works in practice :green_apple:
