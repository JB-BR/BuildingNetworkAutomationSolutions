# Easy Wins

## Credentials strategy
1. Store clear text credentials in a yaml file outside this repository
2. Use those credentials in ansible to get device credentials from a password vault
3. Regularily rotate the clear text credentials (for example once a month)
4. Rotate the credentials in the password vault even more often