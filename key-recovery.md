**Backup recovery algorithm / pseudo code**

Following applies when user has just granted permission to the local app to access inbox. The app will check for backups, and handle any import/recovery in the following way.


```python
# get flat array of all backups, eg [key1a, key2, key1b]
# notice that each backup file can have several armored keys inside of it
armored_key_backups = load_all_armored_keys_from_inbox() 

# group into array of arrays by longid, eg [[key1a, key1b], [key2]]
key_backup_groups = group_keys_by_longid(key_backups)

# unique list of all encountered backup longids
key_backup_longids_all = [key_backup.longid for key_backup in key_backups].unique()

# look through longids that were already imported before. Mark such longids as successfully imported
key_backup_longids_success = key_backup_longids_all.filter(was_key_already_imported_in_the_past)

if key_backup_longids_all.length == 0:
    # no backups
    render_choice_new_key_or_import_key_manually()
else if key_backup_longids_success.length > 0: 
    # at least one key that was backed up is already in the database
    evaluate_recovery_stage_and_render(True)
else:
    # no found backup is in the database yet
    render_found_backups_please_enter_passphrase()


def evaluate_recovery_stage_and_render(at_least_some_backed_up_key_now_present_in_database):
    if not at_least_some_backed_up_key_now_present_in_database:
        # try again
        render_passphrase_did_not_match()
    elif key_backup_longids_success.length == key_backup_longids_all.length:
        # all keys imported (one from each group)
        render_setup_done()  # on most platforms, this means show email inbox
    else:  
        # successfully imported key from at least one group
        # user should try more pass phrases
        render_some_keys_imported_some_still_missing() 


def process_new_passphrase(passphrase):  
    # user just entered a new pass phrase to test
    # test pass pass phrase and import matched keys
    new_key_imported = False
    for key_backup_group in key_backup_groups:
        for key in key_backup_group:
            if key.longid in key_backup_longids_success:
                break  # skip to next group, we already imported a key with the same longid
            if passphrase_can_unlock_key(passphrase, key.copy()):
                # always test on a copy! Do not save unlocked key into database of keys
                save_new_key_into_database(key)
                key_backup_longids_success.append(key.longid)
                new_key_imported = True
                break  #skip to next group
    evaluate_recovery_stage_and_render(new_key_imported)

```

Fundamentally, the same logic is used in the following situations:
 - cannot decrypt message (missing key) -> import missing backup
 - settings -> keys -> import
