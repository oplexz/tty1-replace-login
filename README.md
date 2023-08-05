
# tty1-replace-login

Original article: [link](https://medium.com/@reefland/replace-the-linux-console-login-with-something-different-ad08ba192dd8)

## Instructions

1. Create a new user

	`sudo useradd conusr -s /bin/rbash -M`
	
2. Lock the new user

	`sudo usermod -L conusr`
	
3. Initialize new user's home directory

	```
	sudo mkdir -p /home/conusr/{.fonts,bin}
	sudo chown conusr /home/conusr/
	```

4. (optional) Download font

	```
	wget -q https://raw.githubusercontent.com/xxxserxxx/gotop/master/fonts/Lat15-VGA16-braille.psf
	sudo mv Lat15-VGA16-braille.psf /home/conusr/.fonts/
	```

5. Pick what's going to be displayed

	The original guide uses btop, but you can replace it with something else. You'll need to install or build your desired software.
	
	1.	btop

		Example (Debian): `sudo apt install btop -y`
		
	2. cbonsai

		Example (Debian): `sudo apt install cbonsai -y`
	
	3. Anything else!

		[ArchWiki](https://wiki.archlinux.org/title/ASCII_art) has a good page on ASCII art software.

6. Create symbolic links

	1. (optional) setfont

		Running the following command assumes you've done step 4.

		`sudo ln -s /usr/bin/setfont /home/conusr/bin`

	2. What you're going to be displaying
	
		1. `which YOURBINARY`

		2. `sudo ln -s FULLPATHFROMWHICH /home/conusr/bin`
		
			(example: `sudo ln -s /usr/games/cbonsai /home/conusr/bin`)

7. Create .profile

	```
	cat << EOF | sudo tee /home/conusr/.profile
	HOME=/home/conusr/
	PATH=/home/conusr/bin
	# Uncomment the next line if you did step 4
	# setfont /home/conusr/.fonts/Lat15-VGA16-braille.psf
	
	# Replace YOURBINARY with binary name from step 6
	YOURBINARY ; exit
	EOF
	```

8. Test if everything works

	`sudo su - conusr`

	Ctrl-C to exit if everything looks fine.

9. Edit the getty service for tty1

	Assuming step 8 went smoothly, you can proceed to edit the getty service for tty1

	`sudo systemctl edit getty@tty1.service`

	This will open your text editor.

	At the very beginning of the file, you will see the following text:

	```
	### Editing /etc/systemd/system/getty@tty1.service.d/override.conf
	### Anything between here and the comment below will become the new contents of the file


	### Lines below this comment will be discarded
	```

	Proceed to paste the following text between "### Anything between here..." and "### Lines below this comment..."

	```
	[Service]
	ExecStart=
	ExecStart=-/sbin/agetty --noissue --autologin conusr %I $TERM
	Type=idle
	```

	After pasting that, save the file and exit your editor.

10. Restart the service

	`sudo systemctl restart getty@tty1.service`

With that, you should be all set!

## Restore login prompt

If you want to restore the login prompt on tty1, run the following commands:

```
sudo rm /etc/systemd/system/getty@tty1.service.d/override.conf
sudo systemctl daemon-reload
sudo systemctl restart getty@tty1.service
```
