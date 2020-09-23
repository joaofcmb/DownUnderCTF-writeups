# In a pickle

- **Category:** Misc
- **Difficulty:** Easy
- **Author:** n00bmaster

We managed to intercept communication between und3rm4t3r and his hacker friends. However it is obfuscated using something. We just can't figure out what it is. Maybe you can help us find the flag?

**Attached file:** [data](./data)

## Writeup

Reading the `data` as a text file, we can see what appears to be unordered fragments of the flag and the following message: `I know that the intelligence agency's are onto me so now i'm using ways to evade them: I am just glad that you know how to use pickle. Anyway the flag is `. This hint and the description of the challenge indicates to a potential tool known as **pickle**.

Doing some quick research, we find out that pickle is a Python module for the serialization of Python objects. Let's try to unpickle (unserialize) the file:

```python
import pickle

data = open("data", 'rb')
flag = pickle.load(data)
data.close()

print(flag)
```

```
{1: 'D', 2: 'UCTF', 3: '{', 4: 112, 5: 49, 6: 99, 7: 107, 8: 108, 9: 51, 10: 95, 11: 121, 12: 48, 13: 117, 14: 82, 15: 95, 16: 109, 17: 51, 18: 53, 19: 53, 20: 52, 21: 103, 22: 51, 23: '}', 24: "I know that the intelligence agency's are onto me so now i'm using ways to evade them: I am just glad that you know how to use pickle. Anyway the flag is "}
```

It looks like we got the flag, with its contents represented as decimal numbers. Let's translate these numbers into characters:

```python
import pickle

data = open("data", 'rb')
flag = pickle.load(data)
data.close()

print(''.join(chr(x) for _, x in flag.items() if isinstance(x, int)))
```

```
p1ckl3_y0uR_m3554g3
```

We've done it, the flag for this challenge is `DUCTF{p1ckl3_y0uR_m3554g3}`.
