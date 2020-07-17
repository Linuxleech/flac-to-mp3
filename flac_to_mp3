#!/usr/bin/env python3

'''convert a range of different audio files to mp3 files
using multiple parallel processes'''

import os
import sys
import subprocess
from multiprocessing.pool import ThreadPool
from multiprocessing import cpu_count
import time
import datetime

def converttomp3(task):
    '''Start up a new ffmpeg subprocess transcode the given audio file
    and save the newly transcoded file to a directory within the same
    directory of the original audio '''
    root_path = task[0]
    filename = task[1]
    full_path = os.path.join(root_path, filename)
    new_filename = os.path.splitext(filename)[0] +".mp3"
    new_path = os.path.join(root_path,
                            os.path.basename(root_path) + "-" + FOLDER_NAME,
                            new_filename)

    completed = subprocess.run(["ffmpeg",
                                "-loglevel",
                                "quiet",
                                "-hide_banner",
                                "-y",
                                "-i",
                                full_path,
                                "-write_id3v1",
                                "1",
                                "-id3v2_version",
                                "3",
                                "-codec:a",
                                "libmp3lame",
                                "-q:a",
                                "3",
                                new_path],
                               stderr=subprocess.DEVNULL,
                               stdout=subprocess.DEVNULL,
                               stdin=subprocess.PIPE)
    #If you don't provide stdin pipe, ffmpeg will not exit gracefully
    #when running multiple instances and will require you to reset your
    # terminal once this script finishes executing
    #Remove the original files once they have been transcoded
    #if completed.returncode == 0:
        #subprocess.call(["rm", full_path]) # remove the original file once transcoded
    print(f"'{new_path}' - return code {completed.returncode}")
    if completed.returncode != 0:
        completed.timestamp = datetime.datetime.now().ctime()
    return completed

if __name__ == "__main__":
    FOLDERS = []
    FOLDER_NAME = "MP3s"
    AUDIO_FILE_TYPES = ("flac",
                        "aac",
                        "aiff",
                        "m4a",
                        "ogg",
                        "opus",
                        "raw",
                        "wav",
                        "wma",
                        "webm")
    STARTTIME = time.time()
    #get all of the source audio filenames
    for root, dirs, files in os.walk(os.getcwd()):
        source_audio_filenames = []
        for file in files:
            if file.endswith(AUDIO_FILE_TYPES):
                source_audio_filenames.append((root, file))
        FOLDERS.append((root, source_audio_filenames))

    with ThreadPool(cpu_count())as p:
        PROCESSES = []
        for Folder in FOLDERS:
            try:
                #Stop directories being created within the output directories
                if FOLDER_NAME in Folder[0]:
                    continue
                NewFolderName = os.path.basename(Folder[0]) + "-" + FOLDER_NAME
                os.mkdir(os.path.join(Folder[0], NewFolderName))
            except FileExistsError:
                pass
            PROCESSES += Folder[1]
        print(f"Transcoding {len(PROCESSES)} Audio files")
        JOBS = p.map(converttomp3, PROCESSES)
        FAILED_JOBS = []
        for job in JOBS:
            if job.returncode != 0:
                FAILED_JOBS.append(job)
        MESSAGE = (f"Transcode Finished! \r {len(PROCESSES)-len(FAILED_JOBS)}/{len(PROCESSES)} "
                   f"Audio files transcoded in \r{time.time() - STARTTIME:.4f} seconds")
        try:
            subprocess.run(["notify-send", "--urgency=low", MESSAGE])
        except FileNotFoundError:
            pass
        
        if len(FAILED_JOBS) > 0:       
            with open("nautilus-transcode.log", 'a+') as f:
                for failedJob in FAILED_JOBS:
                    f.write(f"{failedJob.timestamp} args:{failedJob.args}"
                            f"return code:{failedJob.returncode}\n")
    print("Done")
    sys.exit(0)
