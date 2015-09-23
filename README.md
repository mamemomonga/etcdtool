# Etcd-Export/Import/Validate

Export/Import Etcd directory as JSON/YAML/TOML and validate directory using JSON schema.

# Use cases

- Backup/Restore in a format which is not database or version specific.
- Migration of data from production to testing.
- Store authorative configuration in Git and use etcd-import to populate Etcd.
- Copy data from one directory to another.
- Validate directory entries using JSON schema.

# Caveats

- Etcd doesn't support list's, this is handled by using the index as the key:

**JSON Input:**

```json
{
    "users": [
        { "username": "jblack", "first_name": "John", "last_name": "Blackbeard" },
        { "username": "ltrier", "first_name": "Lars", "last_name": "Von Trier" }
    ]
}
```      

**Result in Etcd:**

```
users/0/username: jblack
users/0/first_name: John
users/0/last_name: Blackbeard
users/1/username: ltrier
users/1/first_name: Ludwig
users/1/last_name: Von Treimer
```

# Usage Etcd Export

```bash
Usage of bin/etcd-export:
  -dir="/": Etcd directory
  -format="JSON": Data serialization format YAML, TOML or JSON
  -node="": Etcd node
  -output="": Output file
  -port="2379": Etcd port
  -version=false: Version
```

> You can also set an env. variable for the Etcd node and port.

```bash
export ETCD_CONN="http://etcd1.example.com:2379"
```

# Usage Etcd Import

```bash
Usage of bin/etcd-import:
  -delete
    	Delete entry before import
  -dir string
    	Etcd directory
  -force
    	Force delete without asking
  -format string
    	Data serialization format YAML, TOML or JSON (default "JSON")
  -input string
    	Input file
  -no-validate
    	No validate using JSON schema
  -node string
    	Etcd node
  -port string
    	Etcd port (default "2379")
  -schema string
    	Etcd key for JSON schema
  -version
    	Version
```

> You can also provide input by using STDIN.

## Example

```
./init-etcd.sh start
eval $(./init-etcd.sh env)
etcdctl mkdir /schemas
etcdctl set /schemas/ntp "$(cat examples/ntpd/schema.json)"
etcdctl mkdir /routes
bin/etcd-import -input examples/ntpd/routes.json -dir /routes -no-validate
etcdctl mkdir /ntp/site1
etcdctl mkdir /ntp/site2
bin/etcd-import -input examples/ntpd/ntpd-site1.json -dir /ntp/site1
bin/etcd-import -input examples/ntpd/ntpd-site2.json -dir /ntp/site2
bin/etcd-export -dir /ntp
```

Skipping validation and validate afterwards.

```bash
bin/etcd-import -input examples/ntpd/ntpd-site2.json -dir /ntp/site2 -no-validate
bin/etcd-validate -dir /ntp -schema /schemas/ntp
```

# Usage Etcd Validate

```bash
Usage of bin/etcd-validate:
  -dir string
    	Etcd directory
  -node string
    	Etcd node
  -port string
    	Etcd port (default "2379")
  -schema string
    	Etcd key for JSON schema
  -version
    	Version
```

# Build

```bash
git clone https://github.com/mickep76/etcd-export.git
cd etcd-export
./build
bin/etcd-export --version
```

# Build RPM

```bash
sudo yum install -y rpm-build
make rpm
sudo rpm -i etcd-export-<version>-<release>.rpm
```

## Test

First configure Docker on your Linux or Mac OS X host.

```bash
./init-etcd.sh start
eval "$(./init-etcd.sh env)"
bin/etcd-import -i example.json
bin/etcd-export -f toml
bin/etcd-export | bin/etcd-import -dir /test
```

# Install using Homebrew on Mac OS X

```bash
brew tap mickep76/funk-gnarge
brew install etcd-export
```
