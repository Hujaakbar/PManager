# Password manager command line Interface

PManager is just a cli password manager that does not include encryption (yet).  Encryption/decryption features can be implemented using libraries/software like [cryptography](https://github.com/pyca/cryptography), [VeraCrypt](https://github.com/veracrypt/VeraCrypt) etc.

In the case of VeraCrypt, create a VeraCrypt volume and store a json file there.
After decrypting the volume, pass the json file path as an argument to the PManager.
When you are done creating, modifying passwords using PManager, unmount the VeraCrypt volume to decrypt it again.

Note:

1. In its current state PManager only uses python standard libraries.
2. Passwords  are stored in a JSON format

## Features & Commands

- [file](#provide-a-json-file-to-be-used-as-a-database)
- [create](#create-an-entry)
- [get](#show-stored-passwords)
- [delete](#delete-an-entry)
- [password](#generate-a-password)
- [passphrase](#generate-a-passphrase)

### Provide a json file to be used as a database

```shell
# provide json file
> python pmanager file desktop/secrets.json

# get the currently stored file path
> python pmanager file
/home/path/to/passwords.json
```

PManager creates a `.pmanager` folder in the user's home directory to store its config file. Inside the config file a path to the json file is stored.
If the config file has been modified more than 30 minutes ago, contents of the config file is cleared to avoid data getting stale.

### Create an entry

```shell
# python pmanager  create <entry details in key=value format>

> python pmanager create name=Google password=123345 url=google.com
```

- Entry details should be provided in `key=value` format. Key name can be anything and as many key value pairs can be provided as desired.
- The first value from key value pairs will be used as an entry name.
- Inside the JSON file, all the entry names will be saved in lowercase.
- PManager automatically adds `created_at` and `updated_at` fields.
- Along with `get` and `delete`, the way `create` command handles **entry name** is case insensitive.

JSON value:

```json
"google": {
        "password": "123345",
        "url": "google.com",
        "created_at": "Mar_05_2025 20:53:38"
    }
```

When you try to "create" an entry that already exists, PManager asks you if you want to update the existing entry.

```shell
> python pmanager  create name=Google password=123345 url=www.google.com notes=demo
 Entry already exists, do you want to update it? [Y]|[N]:
```

Updated entry

```json
"google": {
        "password": "123345",
        "url": "www.google.com",
        "created_at": "Mar_05_2025 20:53:38",
        "notes": "demo",
        "updated_at": "Mar_05_2025 21:00:25"
    }
```

Note `url` field is overridden and `notes` and `updated_at` fields are added. No other field is modified.

### Show stored passwords

PManager provides get command to show entry details.

```shell
# python pmanager get <entry_name>
> python pmanager get google

Google:
        password: 123345
        url: www.google.com
        created_at: Mar_05_2025 20:53:38
        notes: demo
        updated_at: Mar_05_2025 21:00:25
```

`get` command accepts two flags `-a` (`--all`) and `-s` (`--search`).

`-a` flag lists all the entry names.
When `-a` flag is provided, no other argument is accepted.

```shell
> python pmanager get -a

1. Google
2. Yahoo
```

`get` command uses exact match comparison, but You can provide `-s` flag to search for substrings.

```shell
> python pmanager get -s hoo

1. Yahoo
2. Hoover
```

### Delete an entry

`delete <entryname>` command deletes a given entry.

```shell
> python pmanager delete google
To confirm, please enter entry name again:
```

### Generate a password

`generate` command generates a password. By default generated password is 16 character long consisting of 14 upper and lower letters and 2 digits.

Note: By default special characters are not included.

```shell
> python pmanager password
0CzlSftNNx7BWpUO
```

The password lengths, the number of digits and whether to include special characters can be specified using below flags:

- `-l`, `--length` integer  (the password length)

    ```shell
    > python pmanager password -l 20
    geY1mFdiP4olyqyqpUeY
    ```

- `-n`, `--number` integer  (the number of digits to include)

    ```shell
    > python pmanager password -l 20 -n 10
    9M1D6K1B563Ry3sey0w6
    ```

- `-s`, `--special_char`    (whether to include special characters or not)

    ```shell
    > python pmanager password -l 10 -s
    jHD&lH2|<8
    ```

### Generate a passphrase

`passphrase` command generates passphrases.

To generate passphrases linux's words file (`/usr/share/dict/words`) or more memorable [10,000 most commonly used words list](https://github.com/first20hours/google-10000-english) can be used. The file should be specified in the source code.

Generated passphrase can contain between 4-6 words and the last word is surrounded by a digit and a special character.

```shell
> python pmanager passphrase
vacation_julia_spice_hansen_jpg_7Obtain"
```

Using `-min` and `-max` flags, upper and lower boundaries of the passphrase length can be specified.

```shell
> python pmanager passphrase -min 60
tennessee_abstract_sonic_immediate_restructuring_1Flexibility{
```

### Version

`-v` flag outputs the version of the PManager.

```shell
> python pmanager -v
pmanager 1.0
```

## Help messages

PManager includes help message and each command also includes their own helps messages.

Help message of a PManager.

```shell
> python pmanager -h
usage: pmanager [-h] [-v] {create,get,delete,file,password,passphrase} ...

Reads passwords from a specified json-formatted file

positional arguments:
  {create,get,delete,file,password,passphrase}
    create              Creates a new entry
    get                 Show entries
    delete              deletes an entry
    file                set/show path to JSON file
    password            Generates password
    passphrase          Generates a passphrase that also contains digits and special characters

options:
  -h, --help            show this help message and exit
  -v, --version         print current version of pmanager
```

Help message of a command.

```shell
> python pmanager get -h
usage: pmanager get [-h] [-a] [-s] [entry_name]

Shows entries

positional arguments:
  entry_name    show details of specified entry, (exact match)

options:
  -h, --help    show this help message and exit
  -a, --all     show all the entries (entry names only)
  -s, --search  show entry names that includes given substring
```
