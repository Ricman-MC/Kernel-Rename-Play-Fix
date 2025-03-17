# **Changing Kernel Name to Avoid Detection from Play Integrity without kernel compile**  

## **Why This Guide?**  
I was trying to pass **Play Integrity** on my rooted device using **Play Integrity Fix** with Magisk. I followed multiple tutorials and wasted a whole day trying to make it work, but nothing helped. Finally, I checked the **GitHub**, and guess what? **Play Integrity now checks the kernel name, too!**  

Unfortunately, my kernel name was one of the blacklisted ones. So, I flashed the **stock boot.img**, and Play Integrity immediately passed without issues. However, I wanted to keep my **custom kernel** while still passing the check.  

After experimenting, I found that simply **modifying the kernel name** inside the boot image was enough. This guide explains how to do it.  

### Message from PlayIntegrityFix Github:
> If you are using a **custom ROM** or **custom kernel**, ensure that your kernel name **is not blacklisted**. You can check by running:  
> ```sh
> uname -r
> ```
> **List of banned kernel names:** [XDA Forums](https://xdaforums.com/t/module-play-integrity-fix-safetynet-fix.4607985/post-89308909)  

---

## **Requirements**
- Rooted Android device with **TWRP** (or any method to extract boot.img)  
- Linux environment (**Debian**, Ubuntu, might be possible in termux)  
- [`hexedit`](https://packages.debian.org/bookworm/hexedit) installed:  

  ```sh
  sudo apt install hexedit
  ```
- [`Kitchen`](https://github.com/ravindu644/Kitchen) for unpacking/repacking boot.img:  

  ```sh
  git clone https://github.com/ravindu644/Kitchen.git
  ```  
You should be able to use some diferent tool like magisk to unpack the boot.img

---

## **Step 1: Extracting `boot.img`**  
If you have a **TWRP backup**, you can use the file directly after renaming it, if you haven't enabled compression:  
```sh
mv boot.emmc.win boot.img
```
Alternatively, use **`dd`** or any other tool to dump the boot partition:  


---

## **Step 2: Unpacking `boot.img`**  
Navigate to the Kitchen folder and place your `boot.img` inside it:  
```sh
cd Kitchen
bash kitchen unpack boot.img
```
This creates a `workspace/` directory containing the extracted files.  

>![Screenshot_1](https://github.com/user-attachments/assets/0accd985-c457-465b-8543-164e688f7c62)


---

## **Step 3: Finding the Kernel Name**
Inside `workspace/`, you'll see a file named **`kernel`**. We need to find occurrences of the **blacklisted kernel name**:  

```sh
cd workspace
strings kernel | grep -i "ElementalX"
```
Replace `"ElementalX"` with your kernel name blacklisted string.  
My output looked something like this:  

```
Linux version 4.19.110-ElementalX-OP8-2.06 (aaron@bella) ...
/lib/firmware/updates/4.19.110-ElementalX-OP8-2.06
/lib/firmware/4.19.110-ElementalX-OP8-2.06
4.19.110-ElementalX-OP8-2.06
```
The last line is the one we need to modify so copy it also it should match the output from `uname -r`.

---

## **Step 4: Editing the Kernel Name**
Now, we will modify the kernel name using `hexedit`.  

1. Open the `kernel` file in `hexedit`:  

   ```sh
   hexedit kernel
   ```
2. Press **`TAB`** to switch to **ASCII mode**.  
3. Press **`CTRL + S`**, type your kernel name (`4.19.110-ElementalX-OP8-2.06`) , and hit **Enter**.  
4. Modify the name by moving with arrow keys and then typing the letter on spot you want to replace the letter. 
   **The new name must be the same length** (e.g., replace `"ElementalX"` with `"CustomKern"`).  
5. Save changes by pressing **`CTRL + X` → `Y` → `Enter`**.  

>Unedited:  
![Screenshot_2](https://github.com/user-attachments/assets/731e4ecc-f184-4cd3-bf9d-e863fd9efd72)   
Edited:  
![Screenshot_3](https://github.com/user-attachments/assets/0956952c-e8ce-447e-9d7a-8d814650beba)



---

## **Step 5: Repacking `boot.img`**
Navigate back to the Kitchen root folder and repack the image:  
```sh
cd ..
bash kitchen repack
```
This will generate a new boot image:  
```
boot.img-new(signed).img
```

---

## **Step 6: Flashing the New Boot Image**
Copy the modified boot image to your phone:  
```sh
adb push boot.img-new(signed).img /sdcard/
```
Flash it using **TWRP** or **Fastboot**:  
```sh
fastboot flash boot boot.img-new(signed).img
```

---

## **Step 7: Verifying the Kernel Name**
After booting your phone, check if the modification was successful:  
```sh
uname -r
```
If you see your new kernel name (instead of the blacklisted one), **congratulations!** 🎉  

Additionally, test Play Integrity with [**Play Integrity API Checker**](https://play.google.com/store/apps/details?id=gr.nikolasspyr.integritycheck) or one of your liking. If everything was done correctly, your **device integrity should pass!**  

| Successful Play Integrity check | `uname -r` inside Termux showing the modified boot.img |
|--------------------------------|------------------------------------------------------|
| <img src="https://github.com/user-attachments/assets/1b706c7e-a67c-405f-b9ea-31a417d9f94b" height="400"> | <img src="https://github.com/user-attachments/assets/40658819-66a6-4872-abd9-9826af87d101" width="500"> |



---

## **Why Did I Do It This Way?**  
I originally wanted to pass Play Integrity without flashing the stock kernel, because my **OnePlus 8 Pro** had weird behavior with the stock boot image.  

- **Stock boot.img made my phone extremely slow** and increased boot times by 10x. Sometimes, it wouldn't even boot.  
- **Flashing stock boot.img broke WiFi**. Even after restoring the modem firmware, WiFi wouldn't work.  
- **Using a different custom kernel might have fixed it, but I was lazy.** 😆  

So, instead of reinstalling my phone to fix the kernel issue, I just modified the kernel name, and everything worked perfectly.  
You can also compile your own kernel and if you have the skill to do so, do it this might break something but it works for me, no crashes etc.
> Hope this guide helps someone struggling with the same issue! 🚀  

---

### **Credits**
- **Kitchen**: [ravindu644/Kitchen](https://github.com/ravindu644/Kitchen)  
- **Play Integrity Fix**: [GitHub](https://github.com/chiteroman/PlayIntegrityFix)  
