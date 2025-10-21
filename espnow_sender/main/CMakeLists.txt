idf_component_register(
    SRCS "espnow_sender.c"  
    INCLUDE_DIRS "."
    REQUIRES esp_wifi nvs_flash esp_event
)