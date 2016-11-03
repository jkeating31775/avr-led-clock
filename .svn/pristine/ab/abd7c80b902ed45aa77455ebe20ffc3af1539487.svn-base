# ======================================================================
#  Makefile for clock on atmega328p
# ======================================================================
TARGET = clock
SRC = clock.c

# ======================================================================
# You shouldn't have to edit below this line
# ======================================================================
AVRDUDE = avrdude
CC = avr-gcc
OBJCOPY = avr-objcopy

MCU = atmega328p
F_CPU = 16000000

CFLAGS = -Wall -O2 -S -std=gnu99 -mmcu=$(MCU) -I.

# ======================================================================
# change which programmer you are using
# ======================================================================
#AVRDUDE_PROGRAMMER = gpio
AVRDUDE_PROGRAMMER = usbtiny

AVRDUDE_PORT = usb

#AVRDUDE_WRITE_FLASH = -U flash:w:$(TARGET).hex

AVRDUDE_WRITE_FLASH = -U lfuse:w:0xD7:m -U flash:w:$(TARGET).hex

AVRDUDE_FLAGS = -p $(MCU) -c $(AVRDUDE_PROGRAMMER) -v

all:
	$(CC) $(SRC) -o $(TARGET) $(CFLAGS)
	$(OBJCOPY) -O ihex $(TARGET) $(TARGET).hex
	rm -f $(TARGET)

clean:
	rm -f $(TARGET).hex

install: all
	$(AVRDUDE) $(AVRDUDE_FLAGS) $(AVRDUDE_WRITE_FLASH) 

fuse:
	$(AVRDUDE) $(AVRDUDE_FLAGS) -U lfuse:w:0xE7:m -U hfuse:w:0xD9:m -U efuse:w:0x07:m 

.PHONY:   all clean install fuse
