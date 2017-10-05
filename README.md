# hashi-local 
Bringing up the Hashicorp stack on your local computer
(Debian/Ubuntu method)

## Pull the appropriate images from the following pages:

- Consul: https://www.consul.io/downloads.html
- Vault: https://www.vaultproject.io/downloads.html
- Nomad: https://www.nomadproject.io/downloads.html


### Extracting the binaries for use

After downloading the binaries (recommended to download them in /tmp directory),
run the following command to extract them and get them usable:

```
/tmp$ sudo unzip <nameOfDownloadedPackageHere> -d /usr/local/bin
```

Repeat for each of the three binaries.


### Testing for a successful install

For each of the binaries, run the command name followed by "version" to ensure you
got the most recent, updated version.

For example:
```
vault version
```


## Running the Hashi-stack

A few notes about how this will work:

1. Each of the binaries will run in their own terminal and you will be provided with 
a live tail of the logs for each product.  You will need 4 individual terminals.  Once
the individual binaries have started, the log tail will make that terminal unusable for
new commands. Hitting 'ctrl+C' will gracefully shutdown any one of these individual 
processes!

2. In this walk-through, I have purposely avoided automating these processes so that
the user can ascertain the rough components and commands to truly bootstrap a hashistack.



### Bringing up Consul

To bring up a development consul server (non-production, as all defaults are enabled,
making this a super-insecure way to run this...) do the following:

As root, run the following command

```
consul agent -dev -client 0.0.0.0
```

To see the Consul web interface, open your browser and go to the following address:

```
localhost:8500
```

From here you can watch the other Hashicorp products register themselves with Consul.



### Bringing up Vault Server

A typical Vault development server runs with an in-memory datastore. For our purposes,
and to see the full Hashi-stack, we are going to use our dev Consul instance as the 
datastore for our Vault server (Consul be the db for all Vault data).

First, be sure that the file vault-config.hcl exists in the directory you are in. 
(You should be in the directory you pulled from GitHub).  Then run:

```
vault server -config=vault-config.hcl
```

Next, we need to get Vault unsealed.  This can be a bit tricky, so follow along carefully.

Open a new terminal window and run the following command to set the address of Vault
on your local system. (Note: this only persists through the current terminal session!)

```
export VAULT_ADDR='http://127.0.0.1:8200'
```

Now that your Vault cli knows how to address Vault, we need to get Vault prepared. In 
the same terminal in which you set the Vault address, run the following:

```
vault init
```

You should quickly see output similar to the following:

```
Unseal Key 1: <key1>
Unseal Key 2: <key2>
Unseal Key 3: <key3>
Unseal Key 4: <key4>
Unseal Key 5: <key5>
Initial Root Token: <rootToken>

Vault initialized with 5 keys and a key threshold of 3. Please
securely distribute the above keys. When the vault is re-sealed,
restarted, or stopped, you must provide at least 3 of these keys
to unseal it again.

Vault does not store the master key. Without at least 3 keys,
your vault will remain permanently sealed.
```

Yay!  You have initialized a Vault for secure thingz!!

Next, we need to unseal the Vault.  Vault is always sealed when
started to prevent any tampering or immediate infiltration. You
can unseal Vault by using the following sequence.  I have documented
the process below for you to see.  Notice as we enter each key from
the initial list of five, the "key threshold" increases, and with
the final key needed for quorum (3), "Sealed: true" changes to
"Sealed: false".  That means the Vault is unlocked and ready for you
to interact with it!

```
matthew@precision-5520:/tmp$ vault unseal
Key (will be hidden): 
Sealed: true
Key Shares: 5
Key Threshold: 3
Unseal Progress: 1
Unseal Nonce: 2ed93046-66f1-4ff8-811c-038aeb12628c

matthew@precision-5520:/tmp$ vault unseal
Key (will be hidden): 
Sealed: true
Key Shares: 5
Key Threshold: 3
Unseal Progress: 2
Unseal Nonce: 2ed93046-66f1-4ff8-811c-038aeb12628c

matthew@precision-5520:/tmp$ vault unseal
Key (will be hidden): 
Sealed: false
Key Shares: 5
Key Threshold: 3
Unseal Progress: 0
Unseal Nonce:
```

Staying in the same terminal, now run the next command to auth
into the Vault instance as the root user:

```
vault auth
```

It will then prompt you immediately for a token.  The token you 
need was in the initial output of the "vault init" command and
was called the "Initial Root Token".

After entering that you will be fully authenticated and can 
check the status of the Vault server by running:

```
vault status
```

Now, go and check the Consul web UI to see your Vault that has
auto-registered itself and is performing basic health checks
automatically!!

It is important to remember that your Vault server is backed
by Consul, so if you terminate or lose Consul, you will lose
everything in Vault as well and will have to re-setup these
two services.


### Bringing up Nomad

To get Nomad up and running, use: 

      
```
nomad agent -dev
```

You can go to the following address in your browser to see that
Nomad came up completely:

```
localhost:4646
```

You can also run:

```
nomad status
```
