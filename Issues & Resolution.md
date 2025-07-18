### MongoDB Initialization Error on WSL: systemd Absence and Version Conflicts

While I was running `npm run dev --host 0.0.0.0` in Open 5gs webui

I got this error:

`MongooseServerSelectionError: connect ECONNREFUSED 127.0.0.1:27017`

### 🔍 **Root Cause:**

- The app uses **MongoDB** (via Mongoose) at `mongodb://localhost:27017`
    
- But **MongoDB was not running** in your WSL instance
    
- And `systemctl` doesn’t work in WSL (no systemd by default)
    

---

**Quick Resolution Summary**

1. Removed old MongoDB versions:
    

```bash
sudo apt purge -y mongodb* mongo-tools
```

2. Installed MongoDB 5.0 from the official repo:
    

```bash
sudo apt install -y mongodb-org
```

3. Started MongoDB manually (since WSL lacks systemd):
    

```bash
mongod --dbpath /data/db --bind_ip 127.0.0.1 --fork --logpath ~/mongod.log
```

4. Verified MongoDB is running:
    

```bash
ps aux | grep mongod
```

5. Application now connects without `ECONNREFUSED`.




Here’s a full summary of the errors you encountered, **the root cause**, and the **exact file changes** made in `openair1` and `common` directories to resolve them.

---

## ✅ Summary of the Issue

### ❗ Error Message:

```
error: incompatible types when initializing type ‘__m512i’ using type ‘int’
warning: implicit declaration of function ‘_mm512_loadu_epi8’
```

### 🔍 Root Cause:

These errors occurred because of **invalid AVX-512 intrinsics** used in the source code. Specifically:

- `_mm512_loadu_epi8` and `_mm512_storeu_epi8` **do not exist**.
    
- `_mm256_load_epi32` and `_mm256_storeu_epi32` are also **non-existent intrinsics**.
    
- These were likely misused in the code or incorrectly assumed from similar-looking intrinsics.
    

---

## 🧩 Files Changed & Fix Details

### 1. 📄 File: `common/utils/nr/nr_common.c`

**❗ Error:**

```c
__m512i input = _mm512_loadu_epi8(&in[i]);
...
_mm512_storeu_epi8(&out[i], reversed);
```

**🛠️ Fix:**  
Replaced the incorrect intrinsics with the correct AVX-512 load/store functions:

```c
__m512i input = _mm512_loadu_si512((__m512i const *)&in[i]);
...
_mm512_storeu_si512((__m512i *)&out[i], reversed);
```

**💡 Reason:**  
`_mm512_loadu_epi8` does **not exist**. The correct function to load 512-bit (64 bytes) data is `_mm512_loadu_si512`.

---

### 2. 📄 File: `openair1/PHY/NR_TRANSPORT/nr_scrambling.c`

**❗ Error:**

```c
__m256i in_256 = _mm256_load_epi32(&((uint32_t *)in)[i_32]);
__m256i seq_256 = _mm256_load_epi32(&seq[i_32]);
...
_mm256_storeu_epi32(&out[i_32], ...);
```

**🛠️ Fix:**  
Replaced with valid AVX2 intrinsics:

```c
__m256i in_256 = _mm256_loadu_si256((__m256i*)&((uint32_t *)in)[i_32]);
__m256i seq_256 = _mm256_loadu_si256((__m256i*)&seq[i_32]);
...
_mm256_storeu_si256((__m256i*)&out[i_32], ...);
```

**💡 Reason:**  
`_mm256_load_epi32` and `_mm256_storeu_epi32` are **invalid** intrinsics. The correct AVX2 intrinsics are:

- `_mm256_loadu_si256` — for unaligned 256-bit load.
    
- `_mm256_storeu_si256` — for unaligned 256-bit store.
    

---

## 🧪 After Fixing

After these two file corrections:

- The build completed successfully.
    
- All AVX-related errors were eliminated.
    

---

## ✅ Conclusion

|File|Error|Fix Used|
|---|---|---|
|`nr_common.c`|`_mm512_loadu_epi8` not found|Replaced with `_mm512_loadu_si512`|
||`_mm512_storeu_epi8` not found|Replaced with `_mm512_storeu_si512`|
|`nr_scrambling.c`|`_mm256_load_epi32` not found|Replaced with `_mm256_loadu_si256`|
||`_mm256_storeu_epi32` not found|Replaced with `_mm256_storeu_si256`|

Let me know if you want me to generate a patch (`.diff`) for these changes.