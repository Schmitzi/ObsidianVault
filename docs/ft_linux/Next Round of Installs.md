This part is gonna take A LOOOOT of time. Grab a coffee or whatever, sit back and start building everything you need from source. I wish you luck!

## Gettext

```sh
tar -xf gettext-0.26.tar.xz 
cd gettext-0.26

# Configure gettext
./configure --disable-shared

# Then make
make

# Install the msgfmt, msgmerge, and xgettext programs:
cp -v gettext-tools/src/{msgfmt,msgmerge,xgettext} /usr/bin

# Don't forget to clean up after yourself
cd ..
rm -rf gettext-0.26
```