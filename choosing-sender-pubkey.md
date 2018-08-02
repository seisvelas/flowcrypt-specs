# Choosing sender pubkey

This pseudo code is used to decide which public key to use for our own key when sending out a message. 

## Choosing my own pubkey when sending encrypted message
```python
#
# the following code assumes all keys of all accounts are stored in one table, eg `private_keys` that also has a column `armoredPublicKey` of somthing similar.
# additionally it assumes a pre-filled database table `key_emails` with the following structure:
# 
# +-------------+-------------+
# | longid      | userIdEmail |
# +-------------+-------------+
# | DAF456AA..  | em@dom.com  |
# | ......      | ...         |
# +-------------+-------------+
#
# (longid, userIdEmail) combination should have a `UNIQUE KEY` in sql, so that there can be multiple rows with same longid, multiple rows with same userIdEmail but not multiple rows where both (longid, userIdEmail) are the same.
# update the table when new key is imported
# remove from the table by longid if a private key is removed
#

### Searching when sending a message 

If I'm sending from main account address:
1) search in the table by main account address. If found longid, use that key (public version).
2) if not found, show error `No key available for your email address main@email.com.\n\nPlease write human@flowcrypt.com for help.` 

If I'm sending from alias address:
1) search by the alias email. If a key found, use that key.
2) If no key found, search by account email address. If a key was found, use that key.
3) If no key found in step 1) and 2) then show an error: `No key available for your email addresses alias@email.com and main@email.com.\n\nPlease write human@flowcrypt.com for help.` 


# find pubkey based on specific sender email
def find_my_public_key_by_email_address(my_email_address):
  longid = sql("SELECT longid FROM key_emails WHERE userIdEmail = %my_email_address")
  if longid is not None:
    return get_my_own_public_key_by(longid)
  return None


# find pubkey based on account email and send_from address (may or may not be the same)
def choose_my_public_key(account_email, send_from):
  pubkey = find_appropriate_key(send_from)
  if pubkey is not None:
    return pubkey
  if account_email != send_from:  # no key found yet, and sending from alias email
    pubkey = find_appropriate_key(account_email)  # try to find pubkey based on account email
    if pubkey is not None:
      return pubkey
  return None


my_pubkey = choose_my_public_key(account_email, send_from)
if my_pubkey is not None:
  send_email_encrypted_for([my_pubkey, recipient_pubkey_1, ...])
else:
  if account_email == send_from:  # sending from main acct email
    show_error('No key available for your email address %account_email.\n\nPlease write human@flowcrypt.com for help.')
  else:  # sending from alias
    show_error('No key available for your email addresses %send_from and %account_email.\n\nPlease write human@flowcrypt.com for help.')

```
