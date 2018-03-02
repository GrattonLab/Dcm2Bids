# Dcm2Bids

Dcm2Bids helps you to convert DICOM files of a study to [Brain Imaging Data Structure][bids] (BIDS).

Learn more about BIDS and read the [specifications][bids-spec].

Dcm2Bids was originally developed by Christophe Bedetti. This fork has several additional features and is maintained by Johan Carlin.

## Install

The easiest way to get up and running is to clone this repo and run a local pip install:

```
git clone https://github.com/jooh/Dcm2Bids
cd Dcm2Bids
pip install .
```

This adds the installation directory to your PYTHONPATH and the `scripts` directory to your PATH.

#### Software dependencies

- dcm2niix
- nibabel is optional, but useful if you need to override incorrect nifti header information (currently only incorrect RepetitionTime settings)
- pydeface is very useful as a plugin to anonymise anatomicals (see -a input flag)

DICOM to NIfTI conversion is done with `dcm2niix` converter. See their [github][dcm2niix-github] for source or [NITRC][dcm2niix-nitrc] for compiled versions.

## Usage

Although you can access dcm2bids functionality directly through python, we provide a command-line interface which is usually sufficient for standard usage:

```
usage: dcm2bids [-h] -d DICOM_DIR [DICOM_DIR ...] -p PARTICIPANT -c CONFIG
                [-s SESSION] [--clobber] [-n SELECTSERIES [SELECTSERIES ...]]
                [-o OUTPUTDIR] [-l {DEBUG,INFO,WARNING,ERROR,CRITICAL}]
                [-a ANONYMIZER]

optional arguments:
  -h, --help            show this help message and exit
  -d DICOM_DIR [DICOM_DIR ...], --dicom_dir DICOM_DIR [DICOM_DIR ...]
                        DICOM files directory(ies). Wild cards are supported.
  -p PARTICIPANT, --participant PARTICIPANT
                        Participant number in BIDS output
  -c CONFIG, --config CONFIG
                        JSON configuration file (see example/config.json)
  -s SESSION, --session SESSION
                        Session name/number
  --clobber             Overwrite output if it exists
  -n SELECTSERIES [SELECTSERIES ...], --selectseries SELECTSERIES [SELECTSERIES ...]
                        Select subset of series numbers (integers) for
                        conversion
  -o OUTPUTDIR, --outputdir OUTPUTDIR
                        Output BIDS study directory (default current
                        directory)
  -l {DEBUG,INFO,WARNING,ERROR,CRITICAL}, --loglevel {DEBUG,INFO,WARNING,ERROR,CRITICAL}
                        Set logging level (the log file is written to
                        outputdir)
  -a ANONYMIZER, --anonymizer ANONYMIZER
                        Anonymize each anat image by passing it to this shell
                        command (e.g., pydeface.py - the call syntax must be
                        anonymizer inputfile outputfile)
```

dcm2bids should be called -at least- once per session and subject. You are in charge of taking care of iteration over multiple subjects and sessions (if you have more than one per subject). In practice you will probably write a simple shell script that iterates over the scans that make up your dataset, calling dcm2bids on each target folder. The trick is to keep the --outputdir flag the same over all conversions that apply to the same BIDS dataset. dcm2bids will take care of putting output files into the correct sub-folders.

Under the hood, dcm2bids starts by calling dcm2niix with whatever the input -d is. Output niftis and sidecar jsons are written to a temporary directory (tmp_dcm2bids). We then iterate over the converted sequences. Only sequences that meet the criteria for a description in the configuration file will be copied over to the final bids output directory.

dcm2bids initialises the basic files required by the BIDS spec (e.g., the participants.tsv file in the root outputdir). Once you have finished converting dicoms you should probably go over these general files and fill in any additional information (e.g., age and gender for participants.tsv).

## Configuration

You will almost certainly need to create your own configuration file to suit the requirements of your study. Examples are available in the example sub-folder. The basic logic of this file is to provide a list of dictionaries under the 'descriptions' key, the major fields of which are:

* dataType: the BIDS output type for this sequence (e.g., "anat", "func").
* suffix: the BIDS output suffix (e.g., "T1w", "bold").
* criteria: used to match the series that dcm2niix converted with this particular output. For instance `"criteria": {"equal": {"ProtocolName": "3D_EPI_2mm_localiser_B380"}}` ensures that only sequences with this exact ProtocolName value in the nifti header will be chosen. You can specify multiple criteria, in which case only sequences that satisfy all criteria (logical and) are chosen.
* customHeader: dictionary with additional fields to add to the sidecar.json. dcm2niix does a pretty good job with generating the sidecar (make sure you use the latest available version), but it won't be able to get all the fields recommended by the BIDS standard. Also, there are occasional errors that must be manually corrected (especially RepetitionTime, where [dcm2niix reports the MRI physics-correct TR definition while BIDS follows a more psychologically intuitive definition](https://groups.google.com/forum/#!topic/bids-discussion/jPVb-4Ah29A)).

## Troubleshooting

It can take a bit of trial and error to get dcm2bids to recognise your sequences of interest. A few things to consider:

* Inspect the nifti headers of the converted files under tmp_dcm2bids. Any field can be used as a criterion in the configuration file.
* For persistent false positives (e.g., 'bad' EPI runs), consider using the --selectseries input flag to force dcm2bids to only consider a subset of your series.
* If all else fails, you can short-circuit the sidecar parsing functionality by setting -d to the exact series folder that you want to convert (although you will still need to setup a description in your configuration file that matches). You can call dcm2bids multiple times for the same subject and session, so it is possible to work out which series you want to convert in e.g. a shell script and then call dcm2bids separately for each.

[bids]: http://bids.neuroimaging.io/
[bids-spec]: http://bids.neuroimaging.io/#download
[conda]: https://conda.io/docs/
[dcm2niix-github]: https://github.com/rordenlab/dcm2niix
[dcm2niix-nitrc]: https://www.nitrc.org/plugins/mwiki/index.php/dcm2nii:MainPage
