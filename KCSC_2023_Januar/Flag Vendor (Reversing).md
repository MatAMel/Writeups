
## Flag Vendor (Reversing)

In this challenge we were given a binary file which checks the input against a valid license key.  The task was to find out how the binary validated the keys.

The first step was to open the binary in a reversing tool. I used Ghidra which was developed by the NSA.

In Ghidra we can search for strings. I could then see the strings "License key verified" and "flag.txt". These strings were located in the main function.

After Ghidra decompiled main we got this:
Some variable names are changed for better readability.

```c
undefined8 main(void)

{
  FILE *flag_filestream;
  size_t keylen;
  long in_FS_OFFSET;
  int sum;
  int local_bc;
  undefined8 local_b0;
  char input_key [32];
  char flag_content [104];
  long local_20;
  
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  flag_filestream = fopen("flag.txt","r");
  sum = 0;
  ignore_me_init_buffering();
  ignore_me_init_signal();
  puts("Enter a valid license key to purchase a flag:");
  fgets(input_key,0x20,stdin);
  keylen = strlen(input_key);
  local_b0 = keylen - 1;
  if (input_key[keylen - 1] == '\n') {
    input_key[keylen - 1] = '\0';
  }
  keylen = strlen(input_key);
  if (keylen == 0x10) {
    puts("\nValidating...");
    sleep(1);
    local_bc = 0;
    while( true ) {
      keylen = strlen(input_key);
      if (keylen <= (ulong)(long)local_bc) break;
      sum = sum + input_key[local_bc];
      local_bc = local_bc + 1;
    }
    if (sum == 0x539) {
      puts("License key verified!\n");
      sleep(1);
      fgets(flag_content,100,flag_filestream);
      puts(flag_content);
    }
    else {
      puts("Invalid license key!");
    }
  }
  else {
    puts("Invalid license key length.");
  }
  if (local_20 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

The function asks the user to input the key, and it then checks if the key is 0x10 long. 16 in decimal.

So that was my first constraint. The key has to be 16 chars long. 

Then the program looped over the length of the input key and summed up the ASCII-values of the key to a sum variable. 

This sum variable is then compared to the value 0x539 which is 1337 (LEET) in decimal. 

This gave me the second constraint. The ASCII-values of the key has to sum up to 1337 in decimal.

I have done something similar and already had a Python-script in my notes. I just had to make som small adjustemnts, like adding the key length constraint.

```python
import random
import string

ASCII_SUM = 0x539
KEY_LENGTH = 0x10

def check_key(key):
    sum = 0
    for char in key:
        sum += ord(char)
    return sum
    
key = ""
while True:
    key += random.choice(string.ascii_lowercase 
					    + string.ascii_uppercase 
					    + string.digits 
					    + string.punctuation)
    sum = check_key(key)
    if sum > ASCII_SUM:
        key = ""
    elif sum == ASCII_SUM and len(key) == KEY_LENGTH:
        print(f"Found valid key: {key}")
```


Running the script yielded the following output:
```
$ python3 plane.py
Found valid key: @}Pl;b3lc>VY_q(<
Found valid key: hf}5Q_tpyUM''7<I
Found valid key: ^jg+R7'5dmkICmnW
Found valid key: )K!w6lywmgQP5^&m
Found valid key: P<~VoY\66rU.kHLU
Found valid key: au1Q_6D?pmXE)}=l
Found valid key: ocLZ[HFOvK&cW?PY
Found valid key: #PiZ0^e-~h{4_!Yu
Found valid key: Q[*m4pt1QR?\Zu%{
Found valid key: aY?Ju<\<Z(Qo8qqQ
.
.
.
```

By choosing a random key from the output above we would get a key that would give us the flag.
```shell
$  nc 20.240.63.48 30002
Enter a valid license key to purchase a flag:
P<~VoY\66rU.kHLU

Validating...
License key verified!

KCSC{licEns3_k3y_h4cker_guru}
```
