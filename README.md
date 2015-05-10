Analysis
--------

The original executable decrypts and unpacks itself. It can detect a sandboxed environment (e.g. VMWare) and terminates itself. It decrypts its payload (an executable packed with UPX 3.08) using VB5 code, taking 10-20 seconds. It then terminates and creates another process. This executable is written in Delphi.

Functions of interest are found from about 0x495000 to 0x49E000.

It first drops itself in the Windows startup folder. Then it generates a random 10-digit string, the id, and constructs a query string from this, the computer name, a hard-coded number, and the file suffix comprising the id and email address of the malware author. For example:

> number=31&id=4361716884&pc=FOOBAR&tail=.id-4361716884_europay@india.com

In this variant, it is posted to viber-club.ru/script.php and viber-club.ru/wp-admin/script.php. This web host account has since been closed, and the server returns a 403 (Forbidden). It continues to attempt to contact the server, apparently expecting to receive a 32-byte encryption key (possibly AES-256) before it can proceed. It will make no modifications to the computer until it receives a 200 (OK) response.

Even after spoofing the response or bypassing the check, the malware remains inactive, and it ends no a WaitMessage call. If its hidden window is made visible, we see the ransom image and fields resembling those in the unlock.exe tool and the program enters an unresponsive message loop.

There is a further check of the content of a buffer populated from the HTTP response body. If this is empty, it remains dormant. If the check is bypassed, the cryptolocker is activated.

It then finds all files with any of several dozen common extensions that reside in user directories, excluding Program Files, Windows, etc.

It encrypts the first 30,0000 of each file, and stores the number of encrypted bytes in the first 4 bytes, apparently storing those first 4 bytes at the end of the file.

With the key buffer empty, the effective key is 32 zero bytes:

> 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

Filling the key buffer in unlock.exe with the same value successfully decrypts files thus encrypted.

When all files are locked, the wallpaper is set to the ransom image (fud.bmp).

The malware then deletes itself.



Key Recovery
------------

The key is not persisted to disk. If encryption is interrupted, a new id will be generated on the next run, and it continues with a new key retrieved from the server (ignoring already-encrypted files with the .com extension).

It does not seem to be erased from memory, so key recovery may be possible on a machine that has not yet been shutdown since infection.


File Recovery
-------------

[BandarChor](https://www.f-secure.com/weblog/archives/00002795.html) seems to encrypt only the first 30,000 bytes of files so some data recovery is possible without the key. See: https://github.com/tamamshud/BandarChorFileRestorer
