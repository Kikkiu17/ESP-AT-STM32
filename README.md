A library to be able to send and receive ESP AT commands using an STM32. This library uses UART DMA to be able to receive data from the ESP without impacting performance. In the main program loop, incoming data is checked to see if a message from a remote host is incoming.

# Important
**<ins>Enable DMA for UART RX and set it to circular mode. Then you can edit various settings in the `esp8266.h` file.</ins>**

**<ins>Tick the box next to "Generate peripheral initialization as a pair of '.c/.h' files per peripheral" in your project's .ioc file.</ins>**

## How to use
When you run the example, you can connect to the ESP using a TCP client with the port specified in the settings file. The requests are made suing this template: _GET ?key1=value&key2=value_ **OR** _POST ?key1=value&key2=value_

# Example
```
WIFI_t wifi;
Connection_t conn;

int main(void)
{
    if (ESP8266_Init() == TIMEOUT)
    {
        while (1)
        __asm__("nop");
    }

    memcpy(wifi.SSID, ssid, strlen(ssid));
    memcpy(wifi.pw, password, strlen(password));
    WIFI_Connect(&wifi);
    WIFI_SetName(&wifi, (char*)ESP_NAME);
    WIFI_EnableNTPServer(&wifi, 2);

    WIFI_StartServer(&wifi, SERVER_PORT);

    while (1)
    {
        Response_t status = WIFI_ReceiveRequest(&wifi, &conn, AT_SHORT_TIMEOUT);
        if (status == OK)
        {
            char* key_ptr = NULL;

            if ((key_ptr = WIFI_RequestHasKey(&conn, "time")))
            {
                if (WIFI_RequestKeyHasValue(&conn, key_ptr, "now"))
                {
                int32_t hours = WIFI_GetTimeHour(&wifi);
                int32_t minutes = WIFI_GetTimeMinutes(&wifi);
                int32_t seconds = WIFI_GetTimeSeconds(&wifi);
                sprintf(wifi.buf, "%ld:%ld:%ld", hours, minutes, seconds);
                WIFI_SendResponse(&conn, "200 OK", wifi.buf, strlen(wifi.buf));
                }
            }
        }
        // OPTIONAL
        else if (status != TIMEOUT)
        {
            sprintf(wifi.buf, "Status: %d", status);
            WIFI_ResetComm(&wifi, &conn);
            WIFI_SendResponse(&conn, "500 Internal server error", wifi.buf, strlen(wifi.buf));
        }

        // OPTIONAL
        if (!WIFI_response_sent)
        {
            if (status == ERR || status == NULVAL)
                WIFI_ResetComm(&wifi, &conn);
        }
        else
            WIFI_response_sent = false;
    }
}
```
