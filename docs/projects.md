# Projects

Welcome to my GitHub's projects page where I highlight some of the collaborative work I have done, my contributions and some challenging aspects of the projects, oh, and some my solo projects too.

Feel free to use the menu down the side to quickly navigate to a project you find interesting.

TBC...

## Collaborative Projects

### [Atomic Force Microscopy (AFM) Reader](https://github.com/AFM-SPM/AFMReader)

A file reading Python package for propriety atomic force microscopy image files.

<h4>Background</h4>

This was one of the projects I worked on during my PhD that was initially spun-out from the main project TopoStats. This is probably my most downloaded software as it is pivotal in the use of loading AFM images from a wide variety of manufacturers for use within researchers own Python scripts, opposed to the manufacturers limiting analysis software.

Although a few members of the TopoStats team took care of the spin-off package itself, my contribution was the initial build of AFM reader pipeline as file input / output in TopoStats and the `.jpk`, `.ibw`, and `.spm` formats. 

<h4>Why rebuild the wheel?</h4>

This issue arose when we moved away from the open-source Python 2-dependent library "Gwyddion" which loaded many different filetypes and performed AFM image artefact removal. This was necessary to use modern Python 3 libraries in our workflows.

<h4>Design aspects</h4>

This used a factory design pattern where someone should be able to specify an extension to look for, and a switchboard use the appropriate function which returns the same two elements - the image, and the pixel-to-nanometre scaling value.

??? note "Switchboard example"

    ```python
    class LoadFile:
        """
        Class to handle the general loading of an AFM file...
        """

        def __init__(self, filepath: str | Path, channel: str):
            """
            Initialise the general LoadFile class with a filepath and channel...
            """
            self.filepath = Path(filepath)
            self.channel = channel
            self.suffix = self.filepath.suffix

        def load(self) -> tuple[npt.NDArray | str, float | None]:  # noqa: C901
            """
            Generally loads a file type that can be handled by AFMReader...
            """
            if self.suffix == ".asd":
                image, pixel_to_nanometre_scaling_factor, _ = asd.load_asd(self.filepath, self.channel)
            elif self.suffix == ".gwy":
                image, pixel_to_nanometre_scaling_factor = gwy.load_gwy(self.filepath, self.channel)
    ...
    ```

The individual filetype loader modules would then either use an underlying library (`spm` and `ibw`) or code which deconstructs proprietary format (`jpk`). The latter took time to understand that the data was in a type of `tif` format, with data and metadata tags which had to be trawled through to find the corresponding keys of the data we needed for the pipeline.

??? note "Tiff trawling example"

    === "Tiff values"

        ```yaml
        jpk:
        n_slots: 32896
        default_slot: 32897
        first_slot_tag: 32912
        first_scaling_type: 32931
        first_scaling_name: 32932
        first_offset_name: 32933
        channel_name: 32848
        trace_retrace: 32849
        grid_ulength: 32834
        grid_vlength: 32835
        grid_ilength: 32838
        grid_jlength: 32839
        slot_size: 48
        ```

    === "Loading code"
        
        ```python
        def load_jpk(
            file_path: Path | str, channel: str, config_path: Path | str | None = None, flip_image: bool | None = True
        ) -> tuple[np.ndarray, float]:
            """
            Load image from JPK Instruments .jpk files.
            """
            file_path = Path(file_path)
            filename = file_path.stem
            jpk_tags = _load_jpk_tags(config_path)
            try:
                tif = tifffile.TiffFile(file_path)
            except FileNotFoundError: raise
            # Obtain channel list for all channels in file
            channel_list = {}
            for i, page in enumerate(tif.pages[1:]):  # [0] is thumbnail
                available_channel = page.tags[jpk_tags["channel_name"]].value  # keys are hexadecimal values
                if page.tags[jpk_tags["trace_retrace"]].value == 0:  # whether img is trace or retrace
                    tr_rt = "trace"
                else:
                    tr_rt = "retrace"
                channel_list[f"{available_channel}_{tr_rt}"] = i + 1
            try:
                channel_idx = channel_list[channel]
            except KeyError as e:
                logger.error(f"'{channel}' not in {file_path.suffix} channel list: {channel_list}")
                raise ValueError(f"'{channel}' not in {file_path.suffix} channel list: {channel_list}") from e
        ```


### [Napari AFM Reader](https://github.com/AFM-SPM/napari-AFMReader)

### [TopoStats](https://github.com/AFM-SPM/TopoStats)

### [Napari TopoStats](https://github.com/AFM-SPM/napari-TopoStats)

### [Python Template](https://github.com/ImperialCollegeLondon/python-template)

### [ProCAT](https://github.com/ImperialCollegeLondon/proCAT)

### [Energy Systems Map](https://github.com/ImperialCollegeLondon/energy-systems-map)

### [ICICLE Benchmarking](https://github.com/ImperialCollegeLondon/icicle_benchmarks)

## Personal Projects

### [TFL Display Board](https://github.com/Max-Gamill/london_map)

### [Portfolio Website (2018)](https://github.com/Max-Gamill/Django-Portfolio)

### [Unslow-mo Videos](https://github.com/Max-Gamill/De-slowmo-videos)
