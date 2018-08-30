#!/usr/bin/env python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals
import argparse
import ctypes
import os
import praw
import platform
import re
import requests
import sys
import time
import shutil
import datetime
import random
from configparser import ConfigParser
from io import StringIO
from collections import defaultdict
import botCredentials

if sys.version_info <= (2, 6):
    import commands as subprocess
else:
    import subprocess

longtermstorageON = False


def load_config():
    
    
    random.seed(datetime.datetime.now())
<<<<<<< HEAD
    subs=["earthporn"]#"wallpapers", "imaginarylandscapes", "earthporn", "spaceporn"]
=======
    subs=["earthporn"] #"wallpapers", "imaginarylandscapes", "earthporn", "spaceporn"]
>>>>>>> 5bd924291baac3b6a11468a0a980a94f32d44ee7
    random.shuffle(subs)
    
    default = defaultdict(str)
    default["subreddit"] = subs[0]
    default["nsfw"] = "False"
    default["time"] = "hour"
    default["display"] = "0"
    default["output"] = "Pictures/Wallpapers"
    

    config_path = os.path.expanduser("~/.config/change_wallpaper_reddit.rc")
    section_name = "root"
    try:
        config = ConfigParser(default)
        with open(config_path, "r") as stream:
            stream = StringIO("[{section_name}]\n{stream_read}".format(section_name=section_name,
                                                                       stream_read=stream.read()))
            if sys.version_info >= (3, 0):
                config.read_file(stream)
            else:
                config.readfp(stream)

            ret = {}

            # Add a value to ret, printing an error message if there is an error
            def add_to_ret(fun, name):
                try:
                    ret[name] = fun(section_name, name)
                except ValueError as e:
                    err_str = "Error in config file.  Variable '{}': {}. The default '{}' will be used."

                    # print sys.stderr >> err_str.format(name, str(e), default[name])
                    ret[name] = default[name]

            add_to_ret(config.get, "subreddit")
            add_to_ret(config.getboolean, "nsfw")
            add_to_ret(config.getint, "display")
            add_to_ret(config.get, "time")
            add_to_ret(config.get, "output")
            
            return ret

    except IOError as e:
        return default

config = load_config()


def parse_args():
    """parse args with argparse
    :returns: args
    """
    parser = argparse.ArgumentParser(description="Daily Reddit Wallpaper")
    parser.add_argument("-s", "--subreddit", type=str, default=config["subreddit"],
                        help="Example: art, getmotivated, wallpapers, ...")
    parser.add_argument("-t", "--time", type=str, default=config["time"],
                        help="Example: new, hour, day, week, month, year")
    parser.add_argument("-n", "--nsfw", action='store_true', default=config["nsfw"], help="Enables NSFW tagged posts.")
    parser.add_argument("-d", "--display", type=int, default=config["display"],
                        help="Desktop display number on OS X (0: all displays, 1: main display, etc")
    parser.add_argument("-o", "--output", type=str, default=config["output"],
                        help="Set the outputfolder in the home directory to save the Wallpapers to.")

    args = parser.parse_args()
    return args


def get_top_image(sub_reddit, sub_name, home_dir, save_dir):
    """Get image link of most upvoted wallpaper of the day
    :sub_reddit: name of the sub reddit
    :return: the image link
    """

    if args.time == "new":
        submissions = sub_reddit.get_new()#limit=10)
    
    else:
        #sub_reddit.top(params={"t": args.time},limit=10)
        submissions = sub_reddit.hot()#limit=10)
        
    for submission in submissions:
        ret = {"id": submission.id}

        save_location = "{home_dir}/{save_dir}/{subreddit}-{id}.jpg".format(home_dir=home_dir, save_dir=save_dir,
                                                                            subreddit=sub_name,
                                                                            id=submission.id)
        print(save_location)
        print(os.path.isfile(save_location))
        if os.path.isfile(save_location) or not (args.nsfw or submission.over_18):
            print("skipping" + str(submission.id))
            continue
        url = submission.url
        # Strip trailing arguments (after a '?')
        url = re.sub(R"\?.*", "", url)
        if url.endswith(".jpg") or url.endswith(".png"):
            ret["url"] = url
            return ret
        # Imgur support
        if ("imgur.com" in url) and ("/a/" not in url) and ("/gallery/" not in url):
            if url.endswith("/new"):
                url = url.rsplit("/", 1)[0]
            id = url.rsplit("/", 1)[1].rsplit(".", 1)[0]
            ret["url"] = "http://i.imgur.com/{id}.jpg".format(id=id)
            return ret


def detect_desktop_environment():
    """Get current Desktop Environment
       http://stackoverflow.com
       /questions/2035657/what-is-my-current-desktop-environment
    :return: environment
    """
    environment = {}
    if os.environ.get("KDE_FULL_SESSION") == "true":
        environment["name"] = "kde"
        environment["command"] = """
                    qdbus org.kde.plasmashell /PlasmaShell org.kde.PlasmaShell.evaluateScript '
                        var allDesktops = desktops();
                        print (allDesktops);
                        for (i=0;i<allDesktops.length;i++) {{
                            d = allDesktops[i];
                            d.wallpaperPlugin = "org.kde.image";
                            d.currentConfigGroup = Array("Wallpaper",
                                                   "org.kde.image",
                                                   "General");
                            d.writeConfig("Image", "file:///{save_location}")
                        }}
                    '
                """
    elif os.environ.get("GNOME_DESKTOP_SESSION_ID"):
        environment["name"] = "gnome"
        environment["command"] = "gsettings set org.gnome.desktop.background picture-uri file://{save_location}"
    elif os.environ.get("DESKTOP_SESSION") == "Lubuntu":
        environment["name"] = "lubuntu"
        environment["command"] = "pcmanfm -w {save_location} --wallpaper-mode=fit"
    elif os.environ.get("DESKTOP_SESSION") == "mate":
        environment["name"] = "mate"
        environment["command"] = "gsettings set org.mate.background picture-filename {save_location}"
    elif os.environ.get("DESKTOP_SESSION") == "i3":
        environment["name"] = "i3"
        environment["command"] = "feh --bg-fill {save_location}"
    else:
        try:
            info = subprocess.getoutput("xprop -root _DT_SAVE_MODE")
            if ' = "xfce4"' in info:
                environment["name"] = "xfce"
        except (OSError, RuntimeError):
            environment = None
            pass
    return environment



def getDirSize(dir):  
	size = 0
	for root, dirs, files in os.walk(dir):  
		size += sum([os.path.getsize(os.path.join(root, name)) for name in files])  
	return size

if __name__ == '__main__':

    args = parse_args()
    subreddit = args.subreddit
    save_dir = args.output
    lts = "E:\media\pics\wallpapers"
    home_dir = os.path.expanduser("~")

    supported_linux_desktop_envs = ["gnome", "mate", "kde", "lubuntu", "i3"]

    # Python Reddit Api Wrapper
    r = praw.Reddit(client_id=botCredentials.clientID, client_secret=botCredentials.clientSecret, user_agent="Get top wallpaper from /r/{subreddit} by /u/ssimunic".format(subreddit=subreddit))

    # Get top image link
    image = get_top_image(r.subreddit(subreddit), home_dir=home_dir, save_dir=save_dir, sub_name=subreddit)
    if "url" not in image:
        sys.exit("Error: No suitable images were found, the program is now" \
                " exiting.")

    # Request image
    response = requests.get(image["url"], allow_redirects=False)

    # If image is available, proceed to save
    if response.status_code == requests.codes.ok:
        # Get home directory and location where image will be saved
        # (default location for Ubuntu is used)
        
        save_location = "{home_dir}/{save_dir}/{subreddit}-{id}.jpg".format(home_dir=home_dir, save_dir=save_dir,
                                                                            subreddit=subreddit,
                                                                            id=image["id"])
        import datetime

        now = datetime.datetime.now()
        save_folder = "{home_dir}/{save_dir}/".format(home_dir=home_dir, save_dir=save_dir)

        dir = os.path.dirname(save_location)
        if not os.path.exists(dir):
            os.makedirs(dir)
            
        print(save_folder)
        size = getDirSize(save_folder)
        print(size)
        if size > 500000000:
            if longtermstorageON:
                print ("moving folder to lts")
                savename = lts+ "\\" + now.strftime("%Y-%m-%d_%H.%M.%f")
                ltsdir = os.path.dirname(savename + "\\")
                count = 0
                while os.path.exists(ltsdir):
                    savename = savename + str(count)
                    ltsdir = os.path.dirname(savename + "\\")
                    count += 1
                
                shutil.copytree(save_folder, savename)
            shutil.rmtree(save_folder)

        #if os.path.isfile(save_location):
        #    print "Info: Image already exists, nothing to do, the program is" \
        #          " now exiting"

        #    sys.exit("Info: Image already exists, nothing to do, the program is" \
        #          " now exiting")
            
        # Create folders if they don't exist
        dir = os.path.dirname(save_location)
        if not os.path.exists(dir):
            os.makedirs(dir)

        # Write to disk
        with open(save_location, "wb") as fo:
            for chunk in response.iter_content(4096):
                fo.write(chunk)

        # Check OS and environments
        platform_name = platform.system()
        if platform_name.startswith("Lin"):

            # Check desktop environments for linux
            desktop_environment = detect_desktop_environment()
            if desktop_environment and desktop_environment["name"] in supported_linux_desktop_envs:
                os.system(desktop_environment["command"].format(save_location=save_location))
            else:
                print("Unsupported desktop environment")

        # Windows
        if platform_name.startswith("Win"):
            print ("windows detected")
            # Python 3.x
            if sys.version_info >= (3, 0):
                print ("python 3.0")
                ctypes.windll.user32.SystemParametersInfoW(20, 0, save_location, 3)
            # Python 2.x
            else:
                print ("python 2.0")
                ctypes.windll.user32.SystemParametersInfoW(20, 0, save_location, 3)
                #ctypes.windll.user32.SystemParametersInfoA(20, 0, save_location, 3)

        # OS X/macOS
        if platform_name.startswith("Darwin"):
            if args.display == 0:
                command = """
                        osascript -e 'tell application "System Events"
                            set desktopCount to count of desktops
                            repeat with desktopNumber from 1 to desktopCount
                                tell desktop desktopNumber
                                    set picture to "{save_location}"
                                end tell
                            end repeat
                        end tell'
                          """.format(save_location=save_location)
            else:
                command = """osascript -e 'tell application "System Events"
                                set desktopCount to count of desktops
                                tell desktop {display}
                                    set picture to "{save_location}"
                                end tell
                            end tell'""".format(display=args.display,
                                                save_location=save_location)
            os.system(command)
    else:
        sys.exit("Error: Image url is not available, the program is now exiting.")
