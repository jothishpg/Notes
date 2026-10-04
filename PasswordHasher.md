why BCrypt instead of other hashing?

The core distinction: "fast hashes" vs "password hashes"

Algorithms like MD5, SHA-1, SHA-256 are designed to be as fast as possible — they're built for things like verifying a downloaded file wasn't corrupted, or computing a checksum. Speed is a feature there.

For passwords, speed is a liability. Here's why: if an attacker steals your database, they don't try to reverse the hash (that's generally infeasible either way) — they guess millions of candidate passwords, hash each one, and check if it matches. If your hash function can compute a billion hashes per second (SHA-256 on modern GPU hardware genuinely can), an attacker can try billions of password guesses per second against your stolen data. Even a decently complex password could fall within hours.

What BCrypt does differently
BCrypt.gensalt(COST) // COST = 12, meaning 2^12 = 4096 rounds

BCrypt is deliberately slow, and — critically — tunably slow. The cost parameter (here, 12) means the algorithm internally repeats its core computation 2^12 = 4,096 times. This is called a work factor. Instead of computing a hash in microseconds, it takes on the order of hundreds of milliseconds.

$2a$12$N9qo8uLOickgx2ZMRZoMye IjZAgcfl7p92ldGxad68LJZdL17lhWy
\__/\/ \____________________/ \_____________________________/
 |  |          salt (22)                 hash (31)
 |  cost (work factor, 2^12 rounds)
 algorithm version
