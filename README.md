#!/bin/sh

(
# Copyright (C) License 2024 Telegram : @root3rb / 3rb Of The Rooting (root3rb)
echo "\n===== PIFS Random Profile/Fingerprint Picker ====="

if [ ! -d "/data/adb" ]; then
    echo "Can't touch /data/adb - this script needs to run as root!"
    exit 1
fi

if [ ! -d "./JSON" ]; then
    if [ ! -f "./PIFS.zip" ]; then
        echo "Downloading profile/fingerprint repo from GitHub..."
        dUrl="https://raw.githubusercontent.com/mahmoud32xn/magisk-files/master/PIFS-main.zip"
        dTarget="PIFS.zip"
        if [ $(command -v curl) ]; then
            curl -o "$dTarget" "$dUrl"
        elif [ $(command -v wget) ]; then
            wget -O "$dTarget" "$dUrl"
        elif [ $(command -v /system/bin/curl) ]; then
            /system/bin/curl -o "$dTarget" "$dUrl"
        elif [ $(command -v /system/bin/wget) ]; then
            /system/bin/wget -O "$dTarget" "$dUrl"
        elif [ $(command -v /data/data/com.termux/files/usr/bin/curl) ]; then
            /data/data/com.termux/files/usr/bin/curl -o "$dTarget" "$dUrl"
        elif [ $(command -v /data/data/com.termux/files/usr/include/curl) ]; then
            /data/data/com.termux/files/usr/include/curl -o "$dTarget" "$dUrl"
        elif [ $(command -v /data/adb/magisk/busybox) ]; then
            /data/adb/magisk/busybox wget -O "$dTarget" "$dUrl"
        elif [ $(command -v /debug_ramdisk/.magisk/busybox/wget) ]; then
            /debug_ramdisk/.magisk/busybox/wget -O "$dTarget" "$dUrl"
        elif [ $(command -v /sbin/.magisk/busybox/wget) ]; then
            /sbin/.magisk/busybox/wget -O "$dTarget" "$dUrl"
        elif [ $(command -v /system/xbin/wget) ]; then
            /system/xbin/wget -O "$dTarget" "$dUrl"
        elif [ $(command -v /system/xbin/curl) ]; then
            /system/xbin/curl -o "$dTarget" "$dUrl"
        else
            echo "Couldn't find wget or curl to download the repository.\nYou'll have to download it manually."
            exit 1
        fi
    fi
    echo "Extracting profiles/fingerprints from PIFS.zip..."
    unzip -o PIFS.zip -x .git* -x README.md -x LICENSE
    mv ./PIFS-main/JSON .
    mv ./PIFS-main/pickaprint.sh .
    rm -r ./PIFS-main
fi

if [ -v FORCEABI ]; then
    echo "\$FORCEABI is set, will only pick profile from '${FORCEABI}'"
    FList=$(find "./JSON/${FORCEABI}" -type f)
    if [ -z "$FList" ]; then
        echo "No profiles/fingerprints found for ABI list: '${FORCEABI}'."
        exit 2
    fi
else
    echo "Detecting device ABI list..."
    ABIList=$(getprop | grep -E '\[ro\.product\.cpu\.abilist\]: \[' | sed -r 's/\[[^]]+\]: \[(.+)\]/\1/')
    if [ -z "$ABIList" ]; then # Old devices had single string prop for this
        ABIList=$(getprop | grep -E '\[ro\.product\.cpu\.abi\]: \[' | sed -r 's/\[[^]]+\]: \[(.+)\]/\1/')
    fi
    if [ -n "$ABIList" ]; then
        echo "Will use profile/fingerprint with ABI list '${ABIList}'"
        FList=$(find "./JSON/${ABIList}" -type f)
    else
        echo "Couldn't detect ABI list. Will use profile/fingerprint from anywhere."
        FList=$(find ./JSON -type f)
    fi
    if [ -z "$FList" ]; then
        echo "No profiles/fingerprints found for ABI list: '${ABIList}'. Will use profile/fingerprint from anywhere."
        FList=$(find ./JSON -type f)
        if [ -z "$FList" ]; then
            echo "Couldn't find any profiles/fingerprints. Is the $PWD/JSON directory empty?"
            exit 3
        fi
    fi
fi

FCount=$(echo "$FList" | wc -l)
if [ $FCount == 0 ]; then
    echo "Couldn't parse JSON file list!"
    exit 4
fi

echo "Picking a random profile/fingerprint..."
RandFPNum=$((1 + ($RANDOM * 2) % $FCount)) # Get a random index from the list
RandFP=$(echo "$FList" | sed -r "${RandFPNum}q;d") # Get path of random index

echo "\nRandom profile/fingerprint file: '${RandFP/ /}'\n"

echo "Looking for installed PIF module..."
