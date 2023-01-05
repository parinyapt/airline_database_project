# Airline Database Design

## SQL Query Example
#### 1: 
```sql
INSERT INTO `member`(
    `first_name`,
    `middle_name`,
    `last_name`,
    `gender`,
    `date_of_birth`,
    `tel`,
    `email`,
    `password`,
    `create_at`,
    `update_at`
)
VALUES(
    'Parinya',
    NULL,
    'Termkasipanich',
    'M',
    '2002-11-09',
    '66871189903',
    'parinya.prin@g.swu.ac.th',
    '$2a$07$mZUBnOnbC.mWcSzIRMVPF.YopQuq4GQvR/DaiFEc8B4/x9kwVmmZC',
    CURRENT_TIMESTAMP(),
    CURRENT_TIMESTAMP()
);
```

> By Parinya Termkasipanich