## Dump WeChat Messages from Android


WeChat, as the most popular mobile IM app in China, doesn't provide any methods to export structured message history.

We reverse-engineered the storage protocol of WeChat messages, and
provide this tool to decrypt and parse WeChat messages on a rooted android phone.
It can also render the messages into self-contained html files including voice messages, images, emojis, videos, etc.

## How to use:

#### Dependencies:
+ adb and rooted android phone connected to a Linux/Mac OSX/Win10+Bash.
  If the phone does not come with adb support, you can download an app such as https://play.google.com/store/apps/details?id=eu.chainfire.adbd
+ Python >= 3.6
+ sqlcipher >= 4.1
+ sox (command line tools)
+ Silk audio decoder (included; build it with `./third-party/compile_silk.sh`)
+ Other python dependencies: `pip install -r requirements.txt`.

#### Get Necessary Data:

1. Pull database file and (for older wechat versions) avatar index:
  + Automatic: `./android-interact.sh db`. It may use an incorrect userid.
  + Manual:
    + Figure out your `${userid}` by inspecting the contents of `/data/data/com.tencent.mm/MicroMsg` on the __root__ filesystem of the device. It should be a 32-character-long name consisting of hexadecimal digits.
    + Get `/data/data/com.tencent.mm/MicroMsg/${userid}/EnMicroMsg.db` from the device.
2. Decrypt database file:
  + Automatic: `./decrypt-db.py decrypt --input EnMicroMsg.db`
  + Manual:
    + Get WeChat uin (an integer), possible ways are:
      + `./decrypt-db.py uin`, which looks for uin in `/data/data/com.tencent.mm/shared_prefs/`
      + Login to web wechat, get wxuin=1234567 from `document.cookie`
    + Get your device id (a positive integer), possible ways are:
      + `./decrypt-db.py imei` implements some ways to find device id.
      + Call `*#06#` on your phone
      + Find IMEI in system settings
    + Decrypt database with combination of uin and device id:

      ```
      ./decrypt-db.py decrypt --input EnMicroMsg.db --imei <device id> --uin <uin>
      ```

      NOTE: you may need to try different ways to get device id and find one that can decrypt the
      database. Some phones may have multiple IMEIs, you may need to try them all.
      The command will dump decrypted database at `EnMicroMsg.db.decrypted`.

  If the above decryption doesn't work, you can also try the [password cracker](https://github.com/chg-hou/EnMicroMsg.db-Password-Cracker)
  to brute-force the key. The encryption key is not very strong.

3. Copy the WeChat user resource directory `/mnt/sdcard/tencent/MicroMsg/${userid}/{avatar,emoji,image2,sfs,video,voice2}` from the phone to the `resource` directory:
	+ `./android-interact.sh res`
	+ Change `RES_DIR` in the script if the location of these directories is different on your phone.
	+ This can take a while. Can be faster to first archive it with `tar` with or without compression, and then copy the archive,
		`busybox tar` is recommended as the Android system's `tar` may choke on long paths.
	+ In the end, we need a `resource` directory with the following subdir: `avatar,emoji,image2,sfs,video,voice2`.


#### Run:
+ Parse and dump text messages of __every__ chat (requires decrypted database):

    ```
    ./dump-msg.py decrypted.db output_dir
    ```

+ List all chats (required decrypted database):

    ```
    ./list-chats.py decrypted.db
    ```

+ Generate statistics report on text messages (requires `output_dir` from `./dump-msg.py`):

    ```
    ./count-message.sh output_dir
    ```

+ Dump messages of one contact to html, containing voice messages, emojis, and images (requires decrypted database and `resource`):

    ```
    ./dump-html.py "<contact_display_name>"
    ```

    The output file is `output.html`.

    Check `./dump-html.py -h` to use different paths.

### Examples:
Screenshots of generated html:

![byvoid](https://github.com/ppwwyyxx/wechat-dump/raw/master/screenshots/byvoid.jpg)

See [here](http://ppwwyyxx.com/static/wechat/example.html) for an example html.
