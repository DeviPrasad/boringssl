
### An exercise in building bssl-sys Rust crate bindings for BoringSSL.

# First successful build on 03-October-2023. Sunday.
# Second successful build on 27-october-2023, Friday.

"bssl-sys" uses "bindgen" as part of the cmake build process to generate Rust compatibility
shims for the targeted platform.

We will build boringssl on x86_64 MBP. We need to do the following first:
## (1) Install bindgen
	
	$ brew install bindgen

	This installed bindgen version 0.66.1

## (2) Install cargo-deny
	$ cargo install --locked cargo-deny


## (3) Determine the Rust target triple 'rust-triple' from https://doc.rust-lang.org/nightly/rustc/platform-support.html#tier-1-with-host-tools
	
	we choose 'x86_64-apple-darwin' as the target.

## (4) Check versions
    $ cmake --version
	cmake version 3.27.7

	$ bindgen --version
	bindgen 0.68.1

	$ cargo --version
	cargo 1.73.0 (9c4383fb5 2023-08-26)

	$ cargo-deny --version
	cargo-deny 0.14.1

	$ rustc --version
	rustc 1.73.0 (cc66ad468 2023-10-03))


## (5) Build boringssl with the following parameters:

	$ rm -rf rust/bssl-crypto/target/

	BORINGSSL_BUILD_DIR=/Users/dprasad/projects/google/boringssl/build
	-DRUST_BINDINGS=x86_64-apple-darwin
	-DCMAKE_BUILD_TYPE=Release

	$ pwd
	/Users/dprasad/projects/google/boringssl
	$ cmake -B build -DRUST_BINDINGS=x86_64-apple-darwin -DCMAKE_BUILD_TYPE=Release
	-- The C compiler identification is AppleClang 14.0.3.14030022
	-- Detecting C compiler ABI info
	-- Detecting C compiler ABI info - done
	-- Check for working C compiler: /Library/Developer/CommandLineTools/usr/bin/cc - skipped
	-- Detecting C compile features
	-- Detecting C compile features - done
	-- The CXX compiler identification is AppleClang 14.0.3.14030022
	-- Detecting CXX compiler ABI info
	-- Detecting CXX compiler ABI info - done
	-- Check for working CXX compiler: /Library/Developer/CommandLineTools/usr/bin/c++ - skipped
	-- Detecting CXX compile features
	-- Detecting CXX compile features - done
	-- Found Perl: /usr/bin/perl (found version "5.30.3") 
	-- The ASM compiler identification is AppleClang
	-- Found assembler: /Library/Developer/CommandLineTools/usr/bin/cc
	-- Performing Test CMAKE_HAVE_LIBC_PTHREAD
	-- Performing Test CMAKE_HAVE_LIBC_PTHREAD - Success
	-- Found Threads: TRUE  
	-- Configuring done (5.0s)
	-- Generating done (0.6s)
	-- Build files have been written to: /Users/dprasad/projects/google/boringssl/build


	$ make -C build

	...
	Linking C static library libcrypto.a
	...
	 Built target ssl
	...
	[ 98%] Built target bssl
	[ 98%] Building CXX object ssl/test/CMakeFiles/bssl_shim.dir/async_bio.cc.o
	[ 98%] Building CXX object ssl/test/CMakeFiles/bssl_shim.dir/bssl_shim.cc.o
	[ 99%] Building CXX object ssl/test/CMakeFiles/bssl_shim.dir/handshake_util.cc.o
	[ 99%] Building CXX object ssl/test/CMakeFiles/bssl_shim.dir/mock_quic_transport.cc.o
	...
	[ 99%] Linking CXX executable bssl_shim
	[ 99%] Built target bssl_shim
	[ 99%] Building C object rust/bssl-sys/CMakeFiles/rust_wrapper.dir/rust_wrapper.c.o

	[ 99%] Linking C static library librust_wrapper.a

### Build rust/bssl-sys
	[ 99%] Built target rust_wrapper
	[100%] Generating wrapper_x86_64-apple-darwin.rs
	[100%] Built target bssl_sys


	BoringSSL static libraries can be found in the "build" directory.

## (6) List static libraries
    $ find ./build/ -name "*.a"
	./build//decrepit/libdecrepit.a
	./build//crypto/libcrypto.a
	./build//ssl/libssl.a
	./build//rust/bssl-sys/librust_wrapper.a
	./build//libpki.a
	./build//libtest_support_lib.a
	./build//libboringssl_gtest.a

## (7) Run tests
	$ go run util/all_tests.go
	...
	All tests passed!

