# Reactive Emails

Reactive emails are a solution to the long-lived problem of spam emails. Spam emails take advantage of one critical flaw of most email addresses: they can't be easily changed.

Reactive emails provide a system for generating unguessable yet recoverable emails that can be changed at any time. The idea is to use a separate reactive email for each service, so that if an email for one service ever gets leaked, you can easily generate a new email address and only need to update it for that one service. In other words, your email addresses are "reactive:" they can be easily changed whenever they get leaked.

## Algorithm

Reactive emails are generated via the following algorithm:

`[Purpose][Email Version Number][Reactive Hash]@[Domain]`

**Purpose:** A word representing the purpose the email is used for. Most of the time, this would be the name of the service you're using the email to register for (e.g. `github`).

**Email Version Number:** The version number of the email. This is needed when an email needs to be regenerated because it got leaked.

**Reactive Hash:** Each email contains an unguessable hash based on the Purpose, the Email Version Number, and a Reactive Hash Secret.

### Reactive Hash

The reactive hash is the core of what makes reactive emails work. The reactive hash is an unguessable 4-letter string that is deterministically generated from every reactive email.

To generate the reactive hash, the SHA256 hash function is used on the combination of the email's purpose, version number, and your reactive hash secret:

```text
SHA256(Purpose + Email Version Number + Reactive Hash Secret)
```

The first 5 characters of this hash represents the email's raw reactive hash, and to convert the raw reactive hash to the final reactive hash, its output is mapped using the following map:

| Character in Raw Reactive Hash | Character in Final Reactive Hash |
| ------------------------------ | -------------------------------- |
| 0                              | b                                |
| 1                              | b                                |
| 2                              | d                                |
| 3                              | d                                |
| 4                              | h                                |
| 5                              | h                                |
| 6                              | m                                |
| 7                              | m                                |
| 8                              | n                                |
| 9                              | n                                |
| a                              | q                                |
| b                              | q                                |
| c                              | v                                |
| d                              | v                                |
| e                              | z                                |
| r                              | z                                |

The purpose of this map is to prevent emails with words or letter combinations resembling words (especially offensive words) from being accidentally generated by the original hash function. The set of characters in the final reactive hash should not form any obvious terms or letter combinations that are deemed unpreferable to appear in email addresses.

## Example

If we're generating a reactive email for a new account on GitHub, our purpose string would be `github`, our version number would be `1`, and our reactive hash secret can be any secret (which should be the same for all reactive emails):

**Purpose:** `github`

**Version Number:** `1`

**Reactive Hash Secret:** `mysecret`

Our raw reactive hash would then look as follows:

```text
Raw Reactive Hash = First 5 characters of SHA256(github1mysecret)
Raw Reactive Hash = 06fc2
```

Using the raw reactive hash character map, our final reactive hash would look like:

```text
Raw Reactive Hash   = 06fc2
Final Reactive Hash = bmzvd
```

And thus, if our email domain is `example.com`, our reactive email address for GitHub would be:

```text
[Purpose][Email Version Number][Reactive Hash]@[Domain]
github1bmzvd@example.com
```

> Note that the email domain must be a custom domain and not a domain like @gmail.com since you need to be able to link all these reactive emails to one email account.

Now, let's say this email accidentally got leaked publically (maybe you accidentally associated one of your public GitHub commits with this reactive email). Now, spammers are unapologetically sending this email tons of spam. Luckily, reactive emails allow you to easily refresh your email by simply bumping the version number and generating a new unguessable reactive hash:

```text
New Raw Reactive Hash = First 5 characters of SHA256(github2mysecret)
New Raw Reactive Hash = 960d7
Final Reactive Hash   = nmbqm
```

Thus, our new GitHub reactive email would be `github2nmbqm@example.com`.

## Explanation

So, why are we using such an involved algorithm to generate reactive emails? Can't we just generate a random unguessable string and have the same effect?

The reason why reactive emails are designed this way is to provide a last-resort in case you lose access to your list of reactive emails. Since reactive emails are not meant to be memorized but rather stored on an email manager, such as a password manager like Bitwarden, it's possible that you may end up losing access to your list of reactive emails (e.g. if you forget your Bitwarden password).

The problem is, many sites depend on your email address for identifying your account. If you lose access to your email, you will end up losing access to many of your accounts with no way to recover them. However, since reactive emails are generated deterministically, you can easily recover the email you used for a certain account as long as you remember the Purpose string you used (which should be the service's name) and your reactive hash secret.
