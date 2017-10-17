# Audio HAL
## 1.HAL 相关基础知识
>参见 [Android中HAL如何向上层提供接口总结](http://blog.csdn.net/myarrow/article/details/7175204)
### a.Hal相关结构体
```c
struct hw_module_t;  
struct hw_module_methods_t;  
struct hw_device_t;  
  
/** 
 * Every hardware module must have a data structure named HAL_MODULE_INFO_SYM 
 * and the fields of this data structure must begin with hw_module_t 
 * followed by module specific information. 
 */  
//每一个硬件模块都每必须有一个名为HAL_MODULE_INFO_SYM的数据结构变量，它的第一个成员的类型必须为hw_module_t  
typedef struct hw_module_t {  
    /* 必须为 HARDWARE_MODULE_TAG */  
    uint32_t tag;  
  
    /** major version number for the module */  
    uint16_t version_major;  
  
    /** minor version number of the module */  
    uint16_t version_minor;  
  
    /** Identifier of module */  
    const char *id;  
  
    /** Name of this module */  
    const char *name;  
  
    /** Author/owner/implementor of the module */  
    const char *author;  
  
    /** Modules methods */  
    //模块方法列表,指向hw_module_methods_t*  
    struct hw_module_methods_t* methods;  
  
    /** module's dso */  
    void* dso;  
  
    /** padding to 128 bytes, reserved for future use */  
    uint32_t reserved[32-7];  
  
} hw_module_t;  

typedef struct hw_module_methods_t {                 //硬件模块方法列表的定义，这里只定义了一个open函数  
	/* Open a specific device 
	注意这个open函数明确指出第三个参数的类型为struct hw_device_t**,此参数会返回给调用者，然后在强制类型转化为 xxx_hw_device ,调用者也就拿到了设备的相关方法和属性
	*/  
    int (*open)(const struct hw_module_t* module, const char* id,
            struct hw_device_t** device);  
} hw_module_methods_t;  
  
/** 
 * Every device data structure must begin with hw_device_t 
 * followed by module specific public methods and attributes. 
每一个设备数据结构 xxx_hw_device 的第一个成员函数必须是hw_device_t类型，其次才是各个公共方法和属性,
 */  
typedef struct hw_device_t {  
    /* 必须为 HARDWARE_DEVICE_TAG */  
    uint32_t tag;  
	...
    /** Close this device */  
    int (*close)(struct hw_device_t* device);  
  
} hw_device_t;
```

----
## 2. QCOM audio hal 相关结构

```C
struct audio_device;
struct audio_hw_device;
struct hw_device_t;

struct audio_device {
    struct audio_hw_device device;
    pthread_mutex_t lock; /* audio_device 的写保护锁 */
    struct mixer *mixer;
    audio_mode_t mode;		// 参见 adev_set_mode（）
    audio_devices_t out_device;
    struct stream_in *active_input;
    struct stream_out *primary_output;
    struct stream_out *voice_tx_output;
    struct stream_out *current_call_output;
    bool bluetooth_nrec;
    bool screen_off;
    int *snd_dev_ref_cnt;
    struct listnode usecase_list;
    struct listnode streams_output_cfg_list;
    struct audio_route *audio_route;
    int acdb_settings;
    bool speaker_lr_swap;
    struct voice voice;				// 通过 voice_init()初始化
    unsigned int cur_hdmi_channels;
    unsigned int cur_wfd_channels;
    bool bt_wb_speech_enabled;

    int snd_card;
    unsigned int cur_codec_backend_samplerate;
    unsigned int cur_codec_backend_bit_width;
    bool is_channel_status_set;
    void *platform;
    unsigned int offload_usecases_state;
    void *visualizer_lib;
    int (*visualizer_start_output)(audio_io_handle_t, int);
    int (*visualizer_stop_output)(audio_io_handle_t, int);
    void *offload_effects_lib;
    int (*offload_effects_start_output)(audio_io_handle_t, int);
    int (*offload_effects_stop_output)(audio_io_handle_t, int);

    struct sound_card_status snd_card_status;
    int (*offload_effects_set_hpx_state)(bool);

    void *adm_data;
    void *adm_lib;
    adm_init_t adm_init;
    adm_deinit_t adm_deinit;
    adm_register_input_stream_t adm_register_input_stream;
    adm_register_output_stream_t adm_register_output_stream;
    adm_deregister_stream_t adm_deregister_stream;
    adm_request_focus_t adm_request_focus;
    adm_abandon_focus_t adm_abandon_focus;

    void (*offload_effects_get_parameters)(struct str_parms *,
                                           struct str_parms *);
    void (*offload_effects_set_parameters)(struct str_parms *);

    bool multi_offload_enable;
    int perf_lock_handle;
    int perf_lock_opts[MAX_PERF_LOCK_OPTS];
    int perf_lock_opts_size;
    bool native_playback_enabled;
};

struct audio_hw_device {
    /**
     * Common methods of the audio device.  This *must* be the first member of audio_hw_device
     * as users of this structure will cast a hw_device_t to audio_hw_device pointer in contexts
     * where it's known the hw_device_t references an audio_hw_device.
     */
    struct hw_device_t common;

    /**
     * used by audio flinger to enumerate what devices are supported by
     * each audio_hw_device implementation.
     *
     * Return value is a bitmask of 1 or more values of audio_devices_t
     *
     * NOTE: audio HAL implementations starting with
     * AUDIO_DEVICE_API_VERSION_2_0 do not implement this function.
     * All supported devices should be listed in audio_policy.conf
     * file and the audio policy manager must choose the appropriate
     * audio module based on information in this file.
     */
    uint32_t (*get_supported_devices)(const struct audio_hw_device *dev);

    /**
     * check to see if the audio hardware interface has been initialized.
	 * returns 0 on success, -ENODEV on failure.
	 * 检测 audio hardware 是否已经初始化
     */
    int (*init_check)(const struct audio_hw_device *dev);

    /** set the audio volume of a voice call. Range is between 0.0 and 1.0 */
    int (*set_voice_volume)(struct audio_hw_device *dev, float volume);

    /**
     * set the audio volume for all audio activities other than voice call.
     * Range between 0.0 and 1.0. If any value other than 0 is returned,
     * the software mixer will emulate this capability.
     */
    int (*set_master_volume)(struct audio_hw_device *dev, float volume);

    /**
     * Get the current master volume value for the HAL, if the HAL supports
     * master volume control.  AudioFlinger will query this value from the
     * primary audio HAL when the service starts and use the value for setting
     * the initial master volume across all HALs.  HALs which do not support
     * this method may leave it set to NULL.
     */
    int (*get_master_volume)(struct audio_hw_device *dev, float *volume);

    /**
	 * set_mode is called when the audio mode changes. 
	 * AUDIO_MODE_NORMAL mode is for standard audio playback, 
	 * AUDIO_MODE_RINGTONE when a ringtone is playing, 
	 * AUDIO_MODE_IN_CALL when a call is in progress.
     */
    int (*set_mode)(struct audio_hw_device *dev, audio_mode_t mode);

    /* mic mute */
    int (*set_mic_mute)(struct audio_hw_device *dev, bool state);
    int (*get_mic_mute)(const struct audio_hw_device *dev, bool *state);

    /* set/get global audio parameters */
    int (*set_parameters)(struct audio_hw_device *dev, const char *kv_pairs);

    /*
     * Returns a pointer to a heap allocated string. The caller is responsible
     * for freeing the memory for it using free().
     */
    char * (*get_parameters)(const struct audio_hw_device *dev,
                             const char *keys);

    /* Returns audio input buffer size according to parameters passed or
     * 0 if one of the parameters is not supported.
     * See also get_buffer_size which is for a particular stream.
     */
    size_t (*get_input_buffer_size)(const struct audio_hw_device *dev,
                                    const struct audio_config *config);

	/** This method creates and opens the audio hardware output stream.
	 * 创造 output stream
     * The "address" parameter qualifies the "devices" audio device type if needed.
     * The format format depends on the device type:
     * - Bluetooth devices use the MAC address of the device in the form "00:11:22:AA:BB:CC"
     * - USB devices use the ALSA card and device numbers in the form  "card=X;device=Y"
     * - Other devices may use a number or any other string.
     */

    int (*open_output_stream)(struct audio_hw_device *dev,
                              audio_io_handle_t handle,
                              audio_devices_t devices,
                              audio_output_flags_t flags,
                              struct audio_config *config,
                              struct audio_stream_out **stream_out,
                              const char *address);

    void (*close_output_stream)(struct audio_hw_device *dev,
                                struct audio_stream_out* stream_out);

    /** This method creates and opens the audio hardware input stream */
    int (*open_input_stream)(struct audio_hw_device *dev,
                             audio_io_handle_t handle,
                             audio_devices_t devices,
                             struct audio_config *config,
                             struct audio_stream_in **stream_in,
                             audio_input_flags_t flags,
                             const char *address,
                             audio_source_t source);

    void (*close_input_stream)(struct audio_hw_device *dev,
                               struct audio_stream_in *stream_in);

	/** 
	* This method dumps the state of the audio hardware 
	* 转储音频设备状态
	*/
    int (*dump)(const struct audio_hw_device *dev, int fd);

    /**
     * set the audio mute status for all audio activities.  If any value other
     * than 0 is returned, the software mixer will emulate this capability.
     */
    int (*set_master_mute)(struct audio_hw_device *dev, bool mute);

    /**
     * Get the current master mute status for the HAL, if the HAL supports
     * master mute control.  AudioFlinger will query this value from the primary
     * audio HAL when the service starts and use the value for setting the
     * initial master mute across all HALs.  HALs which do not support this
     * method may leave it set to NULL.
     */
    int (*get_master_mute)(struct audio_hw_device *dev, bool *mute);

    /**
     * Routing control
     */

    /* Creates an audio patch between several source and sink ports.
     * The handle is allocated by the HAL and should be unique for this
     * audio HAL module. */
    int (*create_audio_patch)(struct audio_hw_device *dev,
                               unsigned int num_sources,
                               const struct audio_port_config *sources,
                               unsigned int num_sinks,
                               const struct audio_port_config *sinks,
                               audio_patch_handle_t *handle);

    /* Release an audio patch */
    int (*release_audio_patch)(struct audio_hw_device *dev,
                               audio_patch_handle_t handle);

    /* Fills the list of supported attributes for a given audio port.
     * As input, "port" contains the information (type, role, address etc...)
     * needed by the HAL to identify the port.
     * As output, "port" contains possible attributes (sampling rates, formats,
     * channel masks, gain controllers...) for this port.
     */
    int (*get_audio_port)(struct audio_hw_device *dev,
                          struct audio_port *port);

    /* Set audio port configuration */
    int (*set_audio_port_config)(struct audio_hw_device *dev,
                         const struct audio_port_config *config);

#ifdef AUDIO_LISTEN_ENABLED
    /** This method creates the listen session and returns handle */
    int (*open_listen_session)(struct audio_hw_device *dev,
                              listen_open_params_t *params,
                              struct listen_session** handle);

    /** This method closes the listen session  */
    int (*close_listen_session)(struct audio_hw_device *dev,
                                struct listen_session* handle);

    /** This method sets the mad observer callback  */
    int (*set_mad_observer)(struct audio_hw_device *dev,
                            listen_callback_t cb_func);

   /**
     *   This method is used for setting listen hal specfic parameters.
     *  If multiple paramets are set in one call and setting any one of them
     *  fails it will return failure.
     */
    int (*listen_set_parameters)(struct audio_hw_device *dev,
                                 const char *kv_pairs);
#endif
};

```

## 3.Audio Stream
### a.相关结构
```c
/*
	也是面向对象的思想，向下继承，父类型放在结构体首位；
*/
struct stream_out;
struct stream_in;
struct audio_stream_out;
struct audio_stream_in;
struct aduio_stream;


struct stream_out {
    struct audio_stream_out stream;
    pthread_mutex_t lock; /* see note below on mutex acquisition order */
    pthread_mutex_t pre_lock; /* acquire before lock to avoid DOS by playback thread */
    pthread_cond_t  cond;
    struct pcm_config config;
    struct compr_config compr_config;
    struct pcm *pcm;
    struct compress *compr;
    int standby;
    int pcm_device_id;
    unsigned int sample_rate;
    audio_channel_mask_t channel_mask;
    audio_format_t format;
    audio_devices_t devices;
    audio_output_flags_t flags;
    audio_usecase_t usecase;
    /* Array of supported channel mask configurations. +1 so that the last entry is always 0 */
    audio_channel_mask_t supported_channel_masks[MAX_SUPPORTED_CHANNEL_MASKS + 1];
    audio_format_t supported_formats[MAX_SUPPORTED_FORMATS+1];
    bool muted;
    uint64_t written; /* total frames written, not cleared when entering standby */
    audio_io_handle_t handle;
    struct stream_app_type_cfg app_type_cfg;

    int non_blocking;
    bool use_small_bufs;
    int playback_started;
    int offload_state;
    pthread_cond_t offload_cond;
    pthread_t offload_thread;
    struct listnode offload_cmd_list;
    bool offload_thread_blocked;

    stream_callback_t offload_callback;
    void *offload_cookie;
    struct compr_gapless_mdata gapless_mdata;
    int send_new_metadata;
    bool send_next_track_params;
    bool is_compr_metadata_avail;
    unsigned int bit_width;

    struct audio_device *dev;
};

/**
 * audio_stream_out is the abstraction interface for the audio output hardware.
 * 音频输出设备的抽象
 * It provides information about various properties（属性） of the audio output
 * hardware driver.
 */
struct audio_stream_out {
    /**
     * Common methods of the audio stream out.  This *must* be the first member of audio_stream_out
     * as users of this structure will cast a audio_stream to audio_stream_out pointer in contexts
     * where it's known the audio_stream references an audio_stream_out.
     */
    struct audio_stream common;

    /**
	 * Return the audio hardware driver estimated latency（估计延迟）） in 
	 * &milliseconds.
     */
    uint32_t (*get_latency)(const struct audio_stream_out *stream);

    /**
     * Use this method in situations where audio mixing is done in the
     * hardware. This method serves as a direct interface with hardware,
     * allowing you to directly set the volume as apposed to via the framework.
     * This method might produce multiple PCM outputs or hardware accelerated
     * codecs, such as MP3 or AAC.
     */
    int (*set_volume)(struct audio_stream_out *stream, float left, float right);

    /**
     * Write audio buffer to driver. Returns number of bytes written, or a
     * negative status_t. If at least one frame was written successfully prior to the error,
     * it is suggested that the driver return that successful (short) byte count
     * and then return an error in the subsequent call.
     *
     * If set_callback() has previously been called to enable non-blocking mode
     * the write() is not allowed to block. It must write only the number of
     * bytes that currently fit in the driver/hardware buffer and then return
     * this byte count. If this is less than the requested write size the
     * callback function must be called when more space is available in the
     * driver/hardware buffer.
     */
    ssize_t (*write)(struct audio_stream_out *stream, const void* buffer,
                     size_t bytes);

    /* return the number of audio frames written by the audio dsp to DAC since
     * the output has exited standby
     */
    int (*get_render_position)(const struct audio_stream_out *stream,
                               uint32_t *dsp_frames);

    /**
     * get the local time at which the next write to the audio driver will be presented.
     * The units are microseconds, where the epoch is decided by the local audio HAL.
     */
    int (*get_next_write_timestamp)(const struct audio_stream_out *stream,
                                    int64_t *timestamp);

    /**
     * set the callback function for notifying completion of non-blocking
     * write and drain.
     * Calling this function implies that all future write() and drain()
     * must be non-blocking and use the callback to signal completion.
     */
    int (*set_callback)(struct audio_stream_out *stream,
            stream_callback_t callback, void *cookie);

    /**
     * Notifies to the audio driver to stop playback however the queued buffers are
     * retained by the hardware. Useful for implementing pause/resume. Empty implementation
     * if not supported however should be implemented for hardware with non-trivial
     * latency. In the pause state audio hardware could still be using power. User may
     * consider calling suspend after a timeout.
     *
     * Implementation of this function is mandatory for offloaded playback.
     */
    int (*pause)(struct audio_stream_out* stream);

    /**
     * Notifies to the audio driver to resume playback following a pause.
     * Returns error if called without matching pause.
     *
     * Implementation of this function is mandatory for offloaded playback.
     */
    int (*resume)(struct audio_stream_out* stream);

    /**
     * Requests notification when data buffered by the driver/hardware has
     * been played. If set_callback() has previously been called to enable
     * non-blocking mode, the drain() must not block, instead it should return
     * quickly and completion of the drain is notified through the callback.
     * If set_callback() has not been called, the drain() must block until
     * completion.
     * If type==AUDIO_DRAIN_ALL, the drain completes when all previously written
     * data has been played.
     * If type==AUDIO_DRAIN_EARLY_NOTIFY, the drain completes shortly before all
     * data for the current track has played to allow time for the framework
     * to perform a gapless track switch.
     *
     * Drain must return immediately on stop() and flush() call
     *
     * Implementation of this function is mandatory for offloaded playback.
     */
    int (*drain)(struct audio_stream_out* stream, audio_drain_type_t type );

    /**
     * Notifies to the audio driver to flush the queued data. Stream must already
     * be paused before calling flush().
     *
     * Implementation of this function is mandatory for offloaded playback.
     */
   int (*flush)(struct audio_stream_out* stream);

    /**
     * Return a recent count of the number of audio frames presented to an external observer.
     * This excludes frames which have been written but are still in the pipeline.
     * The count is not reset to zero when output enters standby.
     * Also returns the value of CLOCK_MONOTONIC as of this presentation count.
     * The returned count is expected to be 'recent',
     * but does not need to be the most recent possible value.
     * However, the associated time should correspond to whatever count is returned.
     * Example:  assume that N+M frames have been presented, where M is a 'small' number.
     * Then it is permissible to return N instead of N+M,
     * and the timestamp should correspond to N rather than N+M.
     * The terms 'recent' and 'small' are not defined.
     * They reflect the quality of the implementation.
     *
     * 3.0 and higher only.
     */
    int (*get_presentation_position)(const struct audio_stream_out *stream,
                               uint64_t *frames, struct timespec *timestamp);

};

/* common audio stream parameters and operations */
struct audio_stream {

    /**
     * Return the sampling rate in Hz - eg. 44100.
     */
    uint32_t (*get_sample_rate)(const struct audio_stream *stream);

    /* currently unused - use set_parameters with key
     *    AUDIO_PARAMETER_STREAM_SAMPLING_RATE
     */
    int (*set_sample_rate)(struct audio_stream *stream, uint32_t rate);

    /**
     * Return size of input/output buffer in bytes for this stream - eg. 4800.
     * It should be a multiple of the frame size.  See also get_input_buffer_size.
     */
    size_t (*get_buffer_size)(const struct audio_stream *stream);

    /**
     * Return the channel mask -
     *  e.g. AUDIO_CHANNEL_OUT_STEREO or AUDIO_CHANNEL_IN_STEREO
     */
    audio_channel_mask_t (*get_channels)(const struct audio_stream *stream);

    /**
     * Return the audio format - e.g. AUDIO_FORMAT_PCM_16_BIT
     */
    audio_format_t (*get_format)(const struct audio_stream *stream);

    /* currently unused - use set_parameters with key
     *     AUDIO_PARAMETER_STREAM_FORMAT
     */
    int (*set_format)(struct audio_stream *stream, audio_format_t format);

    /**
     * Put the audio hardware input/output into standby mode.
     * Driver should exit from standby mode at the next I/O operation.
     * Returns 0 on success and <0 on failure.
     */
    int (*standby)(struct audio_stream *stream);

	/** dump the state of the audio input/output device 
	* 转储音频输入/输出设备的状态
	*/
    int (*dump)(const struct audio_stream *stream, int fd);

    /** Return the set of device(s) which this stream is connected to */
    audio_devices_t (*get_device)(const struct audio_stream *stream);

    /**
     * Currently unused - set_device() corresponds to set_parameters() with key
     * AUDIO_PARAMETER_STREAM_ROUTING for both input and output.
     * AUDIO_PARAMETER_STREAM_INPUT_SOURCE is an additional information used by
     * input streams only.
     */
    int (*set_device)(struct audio_stream *stream, audio_devices_t device);

    /**
     * set/get audio stream parameters. The function accepts a list of
     * parameter key value pairs in the form: key1=value1;key2=value2;...
     *
     * Some keys are reserved for standard parameters (See AudioParameter class)
     *
     * If the implementation does not accept a parameter change while
     * the output is active but the parameter is acceptable otherwise, it must
     * return -ENOSYS.
     *
     * The audio flinger will put the stream in standby and then change the
     * parameter value.
     */
    int (*set_parameters)(struct audio_stream *stream, const char *kv_pairs);

    /*
     * Returns a pointer to a heap allocated string. The caller is responsible
     * for freeing the memory for it using free().
     */
    char * (*get_parameters)(const struct audio_stream *stream,
                             const char *keys);
    int (*add_audio_effect)(const struct audio_stream *stream,
                             effect_handle_t effect);
    int (*remove_audio_effect)(const struct audio_stream *stream,
                             effect_handle_t effect);
};
typedef struct audio_stream audio_stream_t;
```