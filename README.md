# DicomAnonymizer

Python package to anonymize DICOM files.
The anonymization answer to the standard . More information about dicom fields for anonymization can be found [here](http://dicom.nema.org/dicom/2013/output/chtml/part15/chapter_E.html#table_E.1-1).

The default behaviour of this package is to anonymize DICOM fields referenced in [dicomfields](dicomanonymizer/dicomfields.py).

Dicom fields are separated into different groups. Each groups will be anonymized in a different way.

| Group | Action | Action definition |
| --- | --- | --- |
| D_TAGS | replace | Replace with a non-zero length value that may be a dummy value and consistent with the VR** |
| Z_TAGS | empty | Replace with a zero length value, or a non-zero length value that may be a dummy value and consistent with the VR** |
| X_TAGS | delete | Completely remove the tag |
| U_TAGS | replaceUID | Replace all UID's number with a random one in order to keep consistent. Same UID will have the same replaced value |
| Z_D_TAGS | emptyOrReplace | Replace with a non-zero length value that may be a dummy value and consistent with the VR** |
| X_Z_TAGS | deleteOrEmpty | Replace with a zero length value, or a non-zero length value that may be a dummy value and consistent with the VR** |
| X_D_TAGS | deleteOrReplace | Replace with a non-zero length value that may be a dummy value and consistent with the VR** |
| X_Z_D_TAGS | deleteOrEmptyOrReplace | Replace with a non-zero length value that may be a dummy value and consistent with the VR** |
| X_Z_U_STAR_TAGS | deleteOrEmptyOrReplaceUID | If it's a UID, then all numbers are randomly replaced. Else, replace with a zero length value, or a non-zero length value that may be a dummy value and consistent with the VR**|
| ALL_TAGS | | Contains all previous defined tags


# How to build it ?

The sources files can be packaged by using:
`python .\setup.py bdist_wheel`

This command will generate a wheel package in `dist` folder which can be then installed as a python package using
`pip install .\dist\DicomAnonymizer-0.0.1-py2.py3-none-any.whl`

Installing this package will also install an executable named `dicom-anonymizer`. In order to use it, please refer to the next section.



# How to use it ?

This package allows to anonymize a selection of DICOM field (defined or overrided).
The way on how the DICOM fields are anonymized can also be overrided.

**[required]** InputPath = Full path to a single DICOM image or to a folder which contains dicom files
**[required]** OutputPath = Full path to the anonymized DICOM image or to a folder. This folder has to exist.
**[optional]** ActionName = Defined an action name that will be applied to the DICOM tag.
**[optional]** Dictionary = Path to a JSON file which defines actions that will be applied on specific dicom tags (see below)



## Default behaviour

You can use the default anonymization behaviour describe above.

```python
dicom-anonymizer Input Output
```



## Custom rules
You can manually add new rules in order to have different behaviors with certain tags.
This will allow you to override default rules:

**Executable**:
```python
dicom-anonymizer InputFilePath OutputFilePath -t '(0x0001, 0x0001)' ActionName -t '(0x0001, 0x0005)' ActionName2
```
This will apply the `ActionName` to the tag `'(0x0001, 0x0001)'` and `ActionName2` to `'(0x0001, 0x0005)'`

**Note**: ActionName has to be defined in [actions list](#actions-list)

For example, the default behavior of the patient's ID is to be replaced by an empty or null value. If you want to keep this value, then you'll have to run :
```python
python anonymizer.py InputFilePath OutputFilePath -t '(0x0010, 0x0020)' keep
```
This command will override the default behavior executed on this tag and the patient's ID will be kept.



## Custom rules with dictionary file

Instead of having a big command line with several new actions, you can create your own dictionary by creating a json file `dictionary.json` :
```json
{
    "(0x0002, 0x0002)": "ActionName",
    "(0x0003, 0x0003)": "ActionName",
    "(0x0004, 0x0004)": "ActionName",
    "(0x0005, 0x0005)": "ActionName"
}
```
Same as before, the `ActionName` has to be defined in the [actions list](#actions-list).

```python
dicom-anonymizer InputFilePath OutputFilePath --dictionary dictionary.json
```


## Custom/overrides actions

Here is a small example which keep all metadata but update the series description
by adding a suffix passed as a parameter.
```python
import argparse
from dicomanonymizer import *

def main():
    parser = argparse.ArgumentParser(add_help=True)
    parser.add_argument('input', help='Path to the input dicom file or input directory which contains dicom files')
    parser.add_argument('output', help='Path to the output dicom file or output directory which will contains dicom files')
    parser.add_argument('--suffix', action='store', help='Suffix that will be added at the end of series description')
    args = parser.parse_args()

    InputPath = args.input
    OutputPath = args.output

    dictionary = {}

    def setupSeriesDescription(dataset, tag):
        element = dataset.get(tag)
        if element is not None:
            element.value = element.value + '-' + args.suffix

    # ALL_TAGS variable is defined on file dicomfields.py
    # the 'keep' method is already defined into the dicom-anonymizer
    # It will overrides the default behaviour
    for i in ALL_TAGS:
        dictionary[i] = keep

    if args.suffix:
        dictionary[(0x0008, 0x103E)] = setupSeriesDescription

    # Launch the anonymization
    anonymize(InputPath, OutputPath, dictionary)

if __name__ == "__main__":
    main()
```

In your own file, you'll have to define:
- Your custom functions. Be careful, your functions always have in inputs a dataset and a tag
- A dictionary which map your functions to a tag

# Actions list

| Action | Action definition |
| --- | --- |
| empty | Replace with a zero length value, or a non-zero length value that may be a dummy value and consistent with the VR** |
| delete | Completely remove the tag |
| keep | Do nothing on the tag |
| clean | Don't use it for now. This is not implemented |
| replaceUID | Replace all UID's number with a random one in order to keep consistent. Same UID will have the same replaced value |
| emptyOrReplace | Replace with a non-zero length value that may be a dummy value and consistent with the VR** |
| deleteOrEmpty | Replace with a zero length value, or a non-zero length value that may be a dummy value and consistent with the VR** |
| deleteOrReplace | Replace with a non-zero length value that may be a dummy value and consistent with the VR** |
| deleteOrEmplyOrReplace | Replace with a non-zero length value that may be a dummy value and consistent with the VR** |
| deleteOrEmptyOrReplaceUID | If it's a UID, then all numbers are randomly replaced. Else, replace with a zero length value, or a non-zero length value that may be a dummy value and consistent with the VR** |


** VR: Value Representation

Work originally done by Edern Haumont