**Backup recovery algorithm / pseudo code**

Following applies when user has just granted permission to the local app to access inbox. The app will check for backups, and handle any import/recovery in the following way.


```python
# get flat array of all backups, eg [key1a, key2, key1b]
armored_key_backups = load_all_armored_keys_from_inbox() 

# read all keys
key_backups = [crypto_key_read(backup) for backup in armored_key_backups]

# group into array of arrays by longid, eg [[key1a, key1b], [key2]]
key_backup_groups = group_keys_by_longid(key_backups)

# successfully imported key ids
successful_longids = []

# display to user eg "We found {key_count} backups"
key_count = key_backup_groups.length

function process_new_passphrase(passphrase):  # user just entered a new pass phrase to test

    # test pass pass phrase and import matched keys
    new_key_match = False
    for key_backup_group in key_backup_groups:
        for key in key_backup_group:
            if key.longid in successful_longids:
                break  # skip to next group, we already imported a key with the same longid
            if passphrase_can_unlock_key(passphrase, key.copy()):  # always test on a copy! Do not save unlocked key into database of keys
                save_new_key_into_database(key)
                successful_longids.append(key.longid)
                new_key_match = True
                break  #skip to next group

    # evaluate test result
    if new_key_match == False:
        # try again
        render_passphrase_did_not_match()
    else:
        if successful_longids.length == key_count:
            # all keys imported (one from each group)
            render_setup_done()
        else:  
            # successfully imported key from at least one group
            # user should try more pass phrases
            render_some_keys_imported_some_still_missing() 
```
