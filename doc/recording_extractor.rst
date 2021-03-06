Build a RecordingExtractor
----------------------------

Building a new :code:`RecordingExtractor` for a specific file format is as simple as creating a new
subclass based on the predefined base classes provided in the
`spikeextractors <https://github.com/SpikeInterface/spikeextractors>`_ package.

To enable standardization among subclasses, the :code:`RecordingExtractors` is an abstract base class which require a new
subclass to **override all methods which are decorated with @abstractmethod**. The :code:`RecordingExtractors` class has four abstract methods: :code:`get_channel_ids()`, :code:`get_num_frames()`, :code:`get_sampling_frequency()`, and :code:`get_traces()`. So all you need to do is create a class that inherits from :code:`RecordingExtractor` and implements these four methods. 

Along with these four methods, you can also optionally override the :code:`write_recording()` function which enables any :code:`RecordingExtractor` to be written into your format. Also, if you have an implementation of :code:`get_snippets()` that is more efficient that the original implementation, you can optionally override that as well.

Any other methods, such as :code:`set_channel_locations()` or :code:`get_epoch()`, **should not** be overwritten as they are generic functions that any :code:`RecordingExtractor` has access to upon initialization.

Finally, if your file format contains information about the channels (e.g. location, group, etc.), you are suggested to add that as a channel property upon initialization (this is optional).

An example of a RecordingExtractor that adds channel locations is shown `here <https://github.com/SpikeInterface/spikeextractors/blob/master/spikeextractors/extractors/biocamrecordingextractor/biocamrecordingextractor.py>`_.

The contributed extractors are in the **spikeextractors/extractors** folder. You can fork the repo and create a new folder
**myformatextractors** there. In the folder, create a new file named **myformatrecordingextractor.py**.

.. code-block:: python

    from spikeextractors import RecordingExtractor

    class MyFormatRecordingExtractor(RecordingExtractor):
        def __init__(self, file_path, ex_parameter):
            RecordingExtractor.__init__(self)

            ## All file specific initialization code can go here.

            # Important pieces of information include (if available): channel locations, groups, and gains
            # To set these, one can use:
            self.set_channel_locations(locations)  # locations is a np.array (num_channels x 2)
            self.set_channel_groups(groups)  # groups is a list or a np.array with length num_channels
            self.set_channel_gains(gains)  # gains is a list or a np.array with length num_channels

        def get_channel_ids(self):

            # Fill code to get a list of channel_ids. If channel ids are not specified, you can use:
            # channel_ids = range(num_channels)

            return channel_ids

        def get_num_frames(self):

            # Fill code to get the number of frames (samples) in the recordings.

            return num_frames

        def get_sampling_frequency(self, unit_id, start_frame=None, end_frame=None):

            # Fill code to get the sampling frequency of the recordings.

            return sampling_frequency

        def get_traces(self, channel_ids=None, start_frame=None, end_frame=None):
            '''This function extracts and returns a trace from the recorded data from the
            given channels ids and the given start and end frame. It will return
            traces from within three ranges:

                [start_frame, t_start+1, ..., end_frame-1]
                [start_frame, start_frame+1, ..., final_recording_frame - 1]
                [0, 1, ..., end_frame-1]
                [0, 1, ..., final_recording_frame - 1]

            if both start_frame and end_frame are given, if only start_frame is
            given, if only end_frame is given, or if neither start_frame or end_frame
            are given, respectively. Traces are returned in a 2D array that
            contains all of the traces from each channel with dimensions
            (num_channels x num_frames). In this implementation, start_frame is inclusive
            and end_frame is exclusive conforming to numpy standards.

            Parameters
            ----------
            start_frame: int
                The starting frame of the trace to be returned (inclusive).
            end_frame: int
                The ending frame of the trace to be returned (exclusive).
            channel_ids: array_like
                A list or 1D array of channel ids (ints) from which each trace will be
                extracted.

            Returns
            ----------
            traces: numpy.ndarray
                A 2D array that contains all of the traces from each channel.
                Dimensions are: (num_channels x num_frames)
            '''

            # Fill code to get the the traces of the specified channel_ids, from start_frame to end_frame

            return traces

        # optional
        def get_ttl_events(self, start_frame=None, end_frame=None, channel_id=0):
            '''
            Returns an array with frames of TTL signals. To be implemented in sub-classes

            Parameters
            ----------
            start_frame: int
                The starting frame of the ttl to be returned (inclusive)
            end_frame: int
                The ending frame of the ttl to be returned (exclusive)
            channel_id: int
                The TTL channel id

            Returns
            -------
            ttl_frames: array-like
                Frames of TTL signal for the specified channel
            ttl_state: array-like
                State of the transition: 1 - rising, -1 - falling
            '''

            # Fill code to return ttl frames and states

            return ttl_frames, ttl_states

        .
        .
        .
        .
        . #Optional functions and pre-implemented functions that a new RecordingExtractor doesn't need to implement
        .
        .
        .
        .

        @staticmethod
        def write_recording(recording, save_path, other_params):
            '''
            This is an example of a function that is not abstract so it is optional if you want to override it.
            It allows other RecordingExtractor to use your new RecordingExtractor to convert their recorded data into
            your recording file format.
            '''


When you are done you should add your :code:`RecordingExtractor` to the **extarctorlist.py** file. You can optionally write a test in the **tests/test_extractors.py** (this is easier if a
:code:`write_recording` function is implemented).

Finally, make a pull request to the spikeextractor repo, so we can review the code and merge it to the spikeextractors!
