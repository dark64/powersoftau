# Powers of Tau

## Original story

This is a [multi-party computation](https://en.wikipedia.org/wiki/Secure_multi-party_computation) (MPC) ceremony which constructs partial zk-SNARK parameters for _all_ circuits up to a depth of 2<sup>21</sup>. It works by taking a step that is performed by all zk-SNARK MPCs and performing it in just one single ceremony. This makes individual zk-SNARK MPCs much cheaper and allows them to scale to practically unbounded numbers of participants.

This protocol is described in this [paper](https://eprint.iacr.org/2017/1050). It produces parameters for an adaptation of [Jens Groth's 2016 pairing-based proving system](https://eprint.iacr.org/2016/260) using the [BLS12-381](https://github.com/ebfull/pairing/tree/master/src/bls12_381) elliptic curve construction. The security proof relies on a randomness beacon being applied at the end of the ceremony.

## Contributions

Extended to support Ethereum's BN256 curve and made it easier to change size of the ceremony. In addition proof generation process can be done in memory constrained environments now.

---
## To run the ceremony on your machine:

1. Preparation:

```bash
$ rustup update # tested on rustup 1.24.3 and rustc 1.55.0
$ cargo build --release
```

2. Put the `challenge` and `response` file from the previous ceremony to root directory.
3. To generate `new_challenge` run:

```bash
$ cargo run --release --bin verify_transform_constrained challenge response new_challenge 21 256
```

4. Run ceremony:

```bash
$ cargo run --release --bin compute_constrained new_challenge new_response 21 256
```

Put your hash from output response to private gist (example: https://gist.github.com/skywinder/c35ab03c66c6b200b33ea2f388a6df89)

5. Reboot your machine to clean up toxic waste.

6. Save the newly generated `response` file and give it to the next participant.

## Recommendations from original ceremony

Participants of the ceremony sample some randomness, perform a computation, and then destroy the randomness. **Only one participant needs to do this successfully to ensure the final parameters are secure.** In order to see that this randomness is truly destroyed, participants may take various kinds of precautions:

* putting the machine in a Faraday cage
* rebooting the machine afterwards
* rebooting the machine afterwards and disconnecting RAM
* destroying the machine afterwards
* running the software on secure hardware
* not connecting the hardware to any networks
* using multiple machines and randomly picking the result of one of them to use
* using different code than what we have provided
* using a secure operating system
* using an operating system that nobody would expect you to use (Rust can compile to Mac OS X and Windows)
* using an unusual Rust toolchain or [alternate rust compiler](https://github.com/thepowersgang/mrustc)
* lots of other ideas we can't think of

It is totally up to the participants. In general, participants should beware of side-channel attacks and assume that remnants of the randomness will be in RAM after the computation has finished.

## Perpetual Powers of Tau

In this section we are going to use a response from [Perpetual Powers of Tau](https://github.com/privacy-scaling-explorations/perpetualpowersoftau) to prepare for a circuit-specific ceremony (also known as phase 2).

### 1. Pick a response

At the time of writing this, the last response was https://github.com/privacy-scaling-explorations/perpetualpowersoftau/tree/master/0074_daniel_response

### 2. Reduce powers

Perpetual Powers of Tau supports circuits of up to `2^28` (260+ million) constraints. We can reduce the parameters to the power of our choice. In this example, we will reduce the power to `21` (up to 2M constraints).

```bash
$ cargo run --release --bin reduce_powers challenge_0075 challenge_reduced 28 21 256
```

### 3. Apply a random beacon

To finalize the setup, we apply a random beacon to the final challenge. In this example, we are going to use an ethereum block height `16627102` with a corresponding hash `0x171147a580764b8445aa1deaeedf8a81436ca1c9c447612e198cb41376aec3a6`. The process and code for calculating the beacon value should be announced before the block appears.

```bash
$ cargo run --release --bin beacon_constrained challenge_reduced response_final 21 256 171147a580764b8445aa1deaeedf8a81436ca1c9c447612e198cb41376aec3a6 10
```

### 4. Prepare phase 2

```bash
$ cargo run --release --bin prepare_phase2 response_final 21 256
```

This command will generate parameters for various circuit depths which we can use in the [phase 2](https://zokrates.github.io/toolbox/trusted_setup.html).

## License

Licensed under either of

 * Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally
submitted for inclusion in the work by you, as defined in the Apache-2.0
license, shall be dual licensed as above, without any additional terms or
conditions.
