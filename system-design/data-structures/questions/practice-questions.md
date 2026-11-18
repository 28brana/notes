# Data Structures for Scale · Practice Questions

[← Back to Data Structures for Scale](../README.md) · [Q&A with answers →](data-structures-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](data-structures-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**30 questions.**

---

## 01 · Geohash

- [ ] 🟢 Q1. What is a geohash and why is it useful?
- [ ] 🟡 Q2. How do you pick the geohash length for a search?
- [ ] 🟡 Q3. Two users are 5 metres apart but have completely different geohashes. Why, and how do you handle it?
- [ ] 🔴 Q4. Your "nearby drivers" service using geohash is slow in Manhattan but fine elsewhere. What's going on?

---

## 02 · Quad Trees

- [ ] 🟢 Q5. What is a quadtree?
- [ ] 🟡 Q6. Quadtree vs geohash — when would you choose a quadtree?
- [ ] 🔴 Q7. Would you use a quadtree to track 1 million Uber drivers updating every 4 seconds?

---

## 03 · R-Trees

- [ ] 🟢 Q8. What problem does an R-tree solve that a quadtree doesn't handle well?
- [ ] 🟡 Q9. You need "which delivery zone contains this address?" for 10,000 polygon zones. How?

---

## 04 · S2 and H3

- [ ] 🟢 Q10. What are S2 and H3?
- [ ] 🟡 Q11. Why did Uber choose hexagons?
- [ ] 🔴 Q12. Your geohash-based app is expanding to Norway and Iceland, and nearby search behaves oddly there. Why, and what would you change?

---

## 05 · Bloom Filters

- [ ] 🟢 Q13. What is a Bloom filter?
- [ ] 🟢 Q14. Can a Bloom filter give false negatives? False positives?
- [ ] 🟡 Q15. How do Bloom filters speed up Cassandra or RocksDB reads?
- [ ] 🔴 Q16. You built a "username taken" Bloom filter. Now users can delete accounts, and freed names still show as taken. Also, after 2 years, more names show as taken wrongly. What happened?

---

## 06 · Cuckoo Filter

- [ ] 🟢 Q17. What does a Cuckoo filter add over a Bloom filter?
- [ ] 🟡 Q18. What are the downsides of a Cuckoo filter?

---

## 07 · Skip Lists

- [ ] 🟢 Q19. What is a skip list?
- [ ] 🟡 Q20. Why does Redis use a skip list for sorted sets instead of a balanced tree?
- [ ] 🔴 Q21. Design a real-time leaderboard for a game with 50 million players.

---

## 08 · Count-Min Sketch

- [ ] 🟢 Q22. What is a Count-Min Sketch?
- [ ] 🔴 Q23. How would you find the top 10 trending hashtags at 50,000 posts per second?

---

## 09 · HyperLogLog

- [ ] 🟢 Q24. What is HyperLogLog used for?
- [ ] 🟡 Q25. You have daily unique-visitor counts. Can you add them to get weekly uniques?

---

## 10 · Merkle Trees

- [ ] 🟢 Q26. What is a Merkle tree?
- [ ] 🟡 Q27. How do Cassandra or Dynamo use Merkle trees?
- [ ] 🔴 Q28. Two replicas each have 1 billion rows and you suspect a few hundred differ. Copying everything takes hours. What do you do?

---

## 11 · MinHash

- [ ] 🟢 Q29. What is MinHash used for?
- [ ] 🔴 Q30. Spammers repost the same message with small edits and your MD5-based dedup misses them. How do you catch them at scale?
