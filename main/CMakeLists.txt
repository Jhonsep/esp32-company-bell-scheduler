idf_component_register(SRCS "main.c" "clock_service.c" "scheduler.c" "web.c" "recovery.c" "recovery_button.c"
    INCLUDE_DIRS "." EMBED_TXTFILES "index.html"
    REQUIRES bootloader_support driver esp_wifi esp_event esp_netif esp_http_server nvs_flash json mbedtls lwip esp_timer)
