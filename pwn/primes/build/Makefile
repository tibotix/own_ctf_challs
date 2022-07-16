CC=clang
CFLAGS=-Wall -Wl,-z,relro -fstack-protector -fpie -pie -g
OUTDIR=.

primes: primes.c
	mkdir -p $(OUTDIR)
	$(CC) -o $(OUTDIR)/primes $(CFLAGS) primes.c
