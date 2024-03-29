<!--
.. title: Scientific Python on M1 Macbook pro
.. slug: scientific-python-on-m1-macbook-pro.md
.. date: 2021-10-27 02:51:58 UTC-05:00
.. author: Matthias Bussonnier
.. tags: Conda, Apple Silicon, 
.. category:
.. link:
.. description:
.. type: text
.. previewimage: 
-->

For the past five years I've been working on a 2015 intel macbook pro which is starting to show its age. 
I've been pondering getting a new machine as it was starting to get difficult to be on video call and do anything else at the same time. I tried macs with touch bars, but no functions keys was a deal-breaker for me. I was considering the [framework laptop](https://frame.work/), but ended up getting a new 2021 macpro (base model). 

Though it is apple silicon, I know it was going to be likely problematic, so here is my experience getting most of my python stack working on it.

<!-- TEASER_END -->

# The machine itself

Well this is not the main point of the article, but after having it for less than 24h, I'm already used to it, I tried to force myself to use previous models for weeks to no avail. If you want to read more see [this twitter thread](https://twitter.com/Mbussonn/status/1453208684360704000). This also has links to a couple of issues where I'm asked to do some benchmark. Feel free to add to it and I'll do my best. 

# Conda for M1 Chips.

As the CPU is of a different architecture you either need to run conda under intel emulation mode (aka Rosetta), or install a M1 compatible version. 

You _can_ install both conda for M1 and conda for intel and switch between both. There are few instruction to do so. 

## Miniforge for apple silicon

Miniforge on M1 is still experimental and available [here](https://github.com/conda-forge/miniforge).
This would be the closest experience to the classic conda install you are used to on intel macs, though be aware that many packages will not be available. For example don't think of installing pyqt/qt 5., so far only pyqt/qt 6.2 and above will work. This is true both via conda and pip. 


## miniconda/conda via rosetta

This methods means that you will run all of conda under an emulation layer. You will take a performance hit, and might need to understand when/how to turn on/off the emulation layer. 

Even if it may looks straightforward once setup, the error messages you get might not always be the best when trying to get Rosetta to work for the first time. 

In particular if Rosetta is not properly or completely installed nothing will indicate you that.

For me the advice or running silently failed

```
/usr/sbin/softwareupdate --install-rosetta # failed without error code or error message
```

and then the command

```
arch -x86_64 ...
```

Silently ignored the `-x86_64` flag.


[Based on this](https://www.wisdomgeek.com/development/installing-intel-based-packages-using-homebrew-on-the-m1-mac/), I was able to get Rosetta to get it to work.

For me I _had to_ left click and "Duplicate", the "terminal.app", Copy Pasting, or alt-dragging will _not_ have the same effect. Then use "Get info" on the menu and checking "Run under Rosetta", triggered the full installation of Rosetta.

You can check it worked with `arch` without args. 


```zsh
~ % arch
i386
```

Once that is done you can just start a sub shell under emulation with 

```zsh
~ % arch 
arm64
~ % arch -x86_64 zsh
~ % arch
i386
~ % exit
~ % arch
arm64
```

Warning it is a subprocess it's not like conda activate/deactivate.

From within and i386 shell you install and then use conda in the same way you did before, you'll have access to most of the python ecosystem you are used to w/o a hitch. Make sure you activate the i386 shell before.


## GPUs

I don't use GPU much, but if you want to use the mac book pro GPU, you'll have to go with miniforge on apple silicon, I don't believe you can use it when using emulation. 

Following instructions on [image-sc](https://forum.image.sc/t/napari-tensorflow-aicsimageio-stardist-care-n2v-pyclesperanto-running-native-on-apple-silicon-m1/55051) I was able to get my GPU recognise, but still have issue with OpenCl, and CuPy is not yet available.


## Switching between both 

For the projects I'm currently working on I'll probably need to get both a native (Apple silicon), and intel version of conda. 

Fortunately both Miniforge and Miniconda can be installed under different path, which I did. I know have tweaked my `.zshrc` to activate either one or the other conda depending on the current value of `arch.


```
# in .zshrc wrap the lines added by miniconda/miniforge with:
if [ $(arch) = 'i386' ]; then
	...
fi


if [ $(arch) = 'arm64' ]; then
	...
fi

# also define a quick alias to run a single command under i386
alias intel="arch -x86_64"
```


# What's Next

So far I had this computer for less than 24h, and as far as I can tell the available instructions and list of compatible packages are moving fast, so by the time you read this. I think I also need to find how to add the current arch in $PS1 in a not too intrusive manner. 

If switching envs is natural to you, then having two conda install should be fine as well, and I hope that in the end it will be possible to have conda envs that are either arm64 or intel instead of having to switch conda install. 

I have a couple of things that are running under homebrew. It was straitforward to setup, but binary don't seem to be universal, that is to say `arch -x86_84 /opt/brew/bin/bash/` does not seem to work, and as I'll get more`cli tool installed, I might have to separate my i386 and arm64 $PATH.

I'll also try to update this as I've written it in a single pass and i'll discover things later.














