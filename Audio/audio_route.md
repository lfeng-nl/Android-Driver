# Audio Route
## 1. system\media\audio_route\Audio_route.c 相关结构
```c
/*
 *                                           /--> fd
 *                                           |--> snd_ctl_card_info card_info
 *                                           |--> snd_ctl_elem_info elem_info[]
 *                                           |--> mixer_ctl ctl[]
 *              /-> mixer(mixer_open初始化) --\--> count
 *              |
 *  audio_route-|-> num_mixer_ctls
 *              |
 *              |-> mixer_state[ctl_index] --/--> mixer_ctl ctl[] --|--> mixer[] --|--> 
 *              |                            |--> num_values
 *              |                            |--> old_value[num_values]
 *              |                            |--> new_value[num_values]
 *              |                            \--> reset_value[num_values]
                |
 *              |-> mixer_path_size
 *              |-> num_mixer_paths
 *              \-> mixer_path[num_mixer_paths] --/--> mixer_setting setting[length] --/--> ctl_index
 *                                                |                                    |--> num_values
 *                                                |                                    \--> value[num_values]
 *                                                |--> name
 *                                                |--> size（总空间（setting个数））
 *                                                \--> length（现有个数）
 */

struct audio_route {
    struct mixer *mixer;
    unsigned int num_mixer_ctls;
    struct mixer_state *mixer_state;

    unsigned int mixer_path_size;
    unsigned int num_mixer_paths;
    struct mixer_path *mixer_path;
};

struct mixer_state {
    struct mixer_ctl *ctl;
    unsigned int num_values;
    int *old_value;
    int *new_value;
    int *reset_value;
};

struct mixer_path {
    char *name;
    unsigned int size;
    unsigned int length;
    struct mixer_setting *setting;
};

struct mixer {
    int fd;
    struct snd_ctl_card_info card_info;
    struct snd_ctl_elem_info *elem_info;
    struct mixer_ctl *ctl;
    unsigned int count;
};

struct mixer_ctl {
    struct mixer *mixer;
    struct snd_ctl_elem_info *info;
    char **ename;
};

struct mixer_setting {
    unsigned int ctl_index;
    unsigned int num_values;
    int *value;
};
```

