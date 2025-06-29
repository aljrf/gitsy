---
tags:
  - keyer
---
# chording Code

#keyer


```Python
import board
import busio
import time
from digitalio import DigitalInOut, Direction, Pull
import neopixel
from adafruit_debouncer import Debouncer
import usb_hid
from adafruit_hid.keyboard import Keyboard
# Import the keyboard layout description
from adafruit_hid.keyboard_layout_uk import KeyboardLayoutUK
from adafruit_hid.keycode import Keycode
from adafruit_ht16k33 import segments

version = "1.1"
# Make this false for a left-handed keyboard
RIGHT_HANDED = True

class Col:
    # Define color constants for easy reference
    RED = (255, 0, 0)
    YELLOW = (255, 150, 0)
    GREEN = (0, 255, 0)
    CYAN = (0, 255, 255)
    BLUE = (0, 0, 255)
    MAGENTA = (255, 0, 255)
    WHITE = (255, 255, 255)
    BLACK = (0, 0, 0)
    GREY = (10, 10, 10)
    VIOLET = (127,0,155)
    INDIGO = (75,0,130)
    ORANGE = (255,165,0)

    # Tuple of color values for easy iteration
    values=(RED, GREEN, BLUE, YELLOW, MAGENTA, CYAN, GREY, WHITE)

    # Dictionary mapping color values to their names
    names={ RED:"Red", YELLOW:"Yellow", GREEN:"Green", CYAN:"Cyan",
            BLUE:"Blue", MAGENTA:"Magenta", WHITE:"White",BLACK:"Black",
            GREY:"Grey",VIOLET:"Violet",INDIGO:"Indigo",ORANGE:"Orange"}

class Switch:
    def __init__(self, pin, pixel, bit):
        # Initialize a switch with its pin, pixel, and bit value
        self.pin = pin
        self.pixel = pixel
        self.bit = bit

class Key():
    def __init__(self, keyboard, switch):
        # Initialize a key with its associated keyboard and switch
        self.keyboard = keyboard
        self.pixel = switch.pixel
        self.bit = switch.bit
        # Set up the digital I/O for the switch pin with a pull-up resistor
        io = DigitalInOut(switch.pin)
        io.pull = Pull.UP
        # Create a debouncer for the switch to handle rapid changes
        self.debounce = Debouncer(io, interval=0.01)
        self.debounce.update()
        self.pressed = not self.debounce.value
        # Set default colors for the key when pressed and released
        self.down_col = Col.RED
        self.up_col = Col.BLUE
        self.guide_key_col = Col.GREY
        self.guide_key = False

    def update(self):
        # Update the state of the key based on the debouncer
        debounce = self.debounce
        debounce.update()
        if debounce.fell:
            self.key_down()
        if debounce.rose:
            self.key_up()
        # Set the color of the key based on its state
        if self.pressed:
            col = self.down_col
        else:
            col = self.guide_key_col if self.guide_key else self.up_col
        self.set_col(col)

    def set_col(self, col):
        # Set the color of the key's pixel
        self.keyboard.pixels[self.pixel] = col

    def key_down(self):
        # Handle key press event
        keyboard = self.keyboard
        self.pressed = True
        if keyboard.character_bits == 0:
            # Start assembling bits for a new character
            keyboard.assembling_bits = True
        keyboard.character_bits += self.bit

    def key_up(self):
        # Handle key release event
        keyboard = self.keyboard
        self.pressed = False
        bits = keyboard.character_bits
        keyboard.character_bits -= self.bit
        if keyboard.assembling_bits:
            # Finalize the character bits and process the key press
            keyboard.assembling_bits = False
            keyboard.got_bits(bits)

class Processor:
    def __init__(self, keyboard):
        # Initialize a processor with its associated keyboard
        self.keys = keyboard

class TextProcessor(Processor):
    def __init__(self, keyboard):
        super(TextProcessor, self).__init__(keyboard)

    def start(self):
        # Start text processing mode
        self.keys.display_text("")
        self.keys.start_lower_case_text()

    def key_pressed(self, key):
        # Handle key press event in text mode
        print("Key pressed text:", key)
        self.keys.display_text(key)
        self.keys.usb_layout.write(key)

    def update(self):
        # Update method for text processing mode
        pass

class HelpProcessor(Processor):
    def __init__(self, keyboard):
        super(HelpProcessor, self).__init__(keyboard)

    def start(self):
        # Start help mode
        print("Start help")
        self.keys.set_keyboard_state(PicoChord.LOWER_CASE_KEYS)
        self.keys.clear_all_keys()
        self.keys.pixels.show()
        self.keys.scroll_text("Printing help. Hold any key to stop")
        self.mode = PicoChord.HELP_MODE
        time.sleep(2.0)
        self.text_waiting_for_clear_keys = True
        self.test_count = 0

    def key_pressed(self, key):
        # Handle key press event in help mode
        print("Key pressed help:", key)

    def print_stop_check(self):
        # Check if any key is pressed to stop the help mode
        self.keys.update_keys()
        return self.keys.character_bits != 0

    def update(self):
        # Update method for help mode
        if self.text_waiting_for_clear_keys:
            if self.keys.character_bits == 0:
                self.game_waiting_for_clear_keys = False
                self.keys.print_keys()
                self.keys.scroll_text("Help complete")
                self.keys.set_keyboard_state(PicoChord.LOWER_CASE_KEYS)
                self.keys.clear_all_keys()
                self.keys.clear_keyboard_guides()
                self.keys.start_mode(PicoChord.TEXT_MODE)
                self.keys.pixels.show()

class GameProcessor(Processor):
    def __init__(self, keyboard, test_texts):
        super(GameProcessor, self).__init__(keyboard)
        self.keys.game_help_displayed = False
        self.test_texts = test_texts

    def get_test_char(self):
        # Get the current test character for the game
        test_string = self.test_texts[self.game_test_strings_pos]
        return test_string[self.game_txt_pos]

    def game_step_start(self):
        # Start a new game step
        ch = self.get_test_char()
        self.keys.display_text(ch)
        self.game_step_start_time = time.monotonic()
        self.game_help_display_time = self.game_step_start_time + self.game_display_time
        self.keys.clear_keyboard_guides()
        self.keys.game_help_displayed = False

    def game_step_advance_char(self):
        # Advance to the next character in the game
        test_string = self.test_texts[self.game_test_strings_pos]
        self.game_txt_pos += 1
        if self.game_txt_pos == len(test_string):
            self.game_txt_pos = 0
            self.game_test_strings_pos += 1
            if self.game_test_strings_pos == len(self.test_texts):
                self.game_test_strings_pos = 0

    def start(self):
        # Start game mode
        print("Start game")
        self.keys.set_keyboard_state(PicoChord.LOWER_CASE_KEYS)
        self.keys.clear_all_keys()
        self.keys.pixels.show()
        self.keys.scroll_text("Game starting.....")
        time.sleep(2.0)
        self.keys.mode = PicoChord.GAME_MODE
        self.game_display_time = 1.0
        self.game_txt_pos = 0
        self.game_test_strings_pos = 0
        self.game_score = 0
        self.game_waiting_for_clear_keys = True

    def key_pressed(self, key):
        # Handle key press event in game mode
        print("Key pressed game:", key)
        if key == self.get_test_char():
            if self.keys.game_help_displayed:
                self.game_score += 1
            else:
                self.game_score += 5
            self.game_step_advance_char()
            self.game_step_start()
        else:
            score_message = "Game over score " + str(self.game_score)
            self.keys.scroll_text(score_message)
            time.sleep(5)
            self.keys.clear_keyboard_guides()
            self.keys.start_mode(PicoChord.TEXT_MODE)

    def update(self):
        # Update method for game mode
        if self.game_waiting_for_clear_keys:
            if self.keys.character_bits == 0:
                self.game_step_start()
                self.game_waiting_for_clear_keys = False
        else:
            current_time = time.monotonic()
            if not self.keys.game_help_displayed:
                if current_time > self.game_help_display_time:
                    self.keys.display_guide(self.get_test_char())
                    self.keys.game_help_displayed = True

class PicoChord:
    # Define keyboard state constants
    UPPER_CASE_KEYS = 0
    LOWER_CASE_KEYS = 1
    NUMBER_KEYS = 2
    SYMBOL_KEYS = 3

    # Define color mappings for different keyboard states
    keyboard_state_cols = {
        UPPER_CASE_KEYS: Col.YELLOW,
        LOWER_CASE_KEYS: Col.BLUE,
        NUMBER_KEYS: Col.GREEN,
        SYMBOL_KEYS: Col.MAGENTA
    }

    def set_keyboard_state(self, state):
        # Set the keyboard state and update key colors
        self.keyboard_state = state
        new_state_col = PicoChord.keyboard_state_cols[self.keyboard_state]
        for key in self.keys:
            key.up_col = new_state_col

    def get_keyboard_col(self):
        # Get the color for the current keyboard state
        return PicoChord.keyboard_state_cols[self.keyboard_state]

    def do_toggle_caps_lock(self):
        # Toggle caps lock state
        print("Caps lock toggle", self.mode)
        if self.keyboard_state == PicoChord.UPPER_CASE_KEYS:
            self.set_keyboard_state(PicoChord.LOWER_CASE_KEYS)
        elif self.keyboard_state == PicoChord.LOWER_CASE_KEYS:
            self.set_keyboard_state(PicoChord.UPPER_CASE_KEYS)

    def display_text(self, text):
        # Display text on the segment display
        self.display.fill(0)
        self.display.print(text)

    def scroll_text(self, text):
        # Scroll text on the segment display
        self.display.marquee(text, delay=0.25, loop=False)

    TEXT_MODE = 0
    HELP_MODE = 1
    GAME_MODE = 2

    # Methods to start different text behaviors
    def start_lower_case_text(self):
        print("Start lower case text")
        self.set_keyboard_state(PicoChord.LOWER_CASE_KEYS)

    def start_upper_case_text(self):
        print("Start upper case text")
        self.set_keyboard_state(PicoChord.UPPER_CASE_KEYS)

    def start_number_text(self):
        print("Start number text")
        self.set_keyboard_state(PicoChord.NUMBER_KEYS)

    def start_symbol_text(self):
        print("Start symbol text")
        self.set_keyboard_state(PicoChord.SYMBOL_KEYS)

    test_texts = (
        "abcdefghijklmnopqrstuvwxyz",
        "the quick brown fox jumps over the lazy dog",
        "Jackdaws love my big sphinx of quartz.",
        "The five boxing wizards jump quickly.",
        "A Capital Idea",
        "1234567890",
        "I am 21 years old",
        "if a<b print(\"hello\")"
    )

    def check_command(self, bits):
        # Check if the bits correspond to a command and execute it
        if bits in self.command_actions:
            command = self.command_actions[bits]
            name = command[0]
            function = command[1]
            print("Control:", name)
            function(self)

    def got_bits(self, bits):
        # Process the bits to determine the key pressed
        print("Bits:", bits)
        decode = self.state_decodes[self.keyboard_state]
        if bits not in decode:
            self.check_command(bits)
            return
        key = decode[bits]
        if self.keyboard_state == PicoChord.UPPER_CASE_KEYS:
            key = key.upper()
        processor = self.mode_processors[self.mode]
        processor.key_pressed(key)

    def clear_keyboard_guides(self):
        # Clear guide keys on the keyboard
        for key in self.keys:
            key.guide_key = False

    def clear_all_keys(self):
        # Clear all keys on the keyboard
        for key in self.keys:
            col = PicoChord.keyboard_state_cols[self.keyboard_state]
            key.set_col(col)
        self.pixels.show()

    def get_key_bits(self):
        # Get the bit values for the keys based on handedness
        if RIGHT_HANDED:
            bits = (1, 2, 4, 8, 16, 32)
        else:
            bits = (32, 16, 8, 4, 2, 1)
        return bits

    def display_guide(self, ch):
        # Display guide for a character on the keyboard
        char_def = self.lookup_character(ch)
        ch = char_def[0]
        char_bits = char_def[1]
        char_state = char_def[2]
        up_col = self.keyboard_state_cols[char_state]
        pos = 0
        for bit in self.get_key_bits():
            key = self.keys[pos]
            key.up_col = up_col
            key.guide_key = (char_bits & bit) != 0
            pos += 1

    def display_keypress(self, char_def, col):
        # Display key press on the keyboard
        ch = char_def[0]
        char_bits = char_def[1]
        char_state = char_def[2]
        up_col = self.keyboard_state_cols[char_state]
        pos = 0
        for bit in self.get_key_bits():
            key = self.keys[pos]
            key.up_col = up_col
            key.set_col(col if (char_bits & bit) != 0 else key.up_col)
            pos += 1

    def display_char_on_keyboard(self, ch, pressed_col):
        # Display a character on the keyboard
        char_def = self.lookup_character(ch)
        if char_def is not None:
            print('Displaying:', char_def)
            self.display_keypress(char_def, pressed_col)

    def send_animated_text_to_keyboard(self, text):
        # Send animated text to the keyboard
        old_state = self.keyboard_state
        for ch in text:
            self.display_text(ch)
            self.display_char_on_keyboard(ch, self.key_down_col)
            self.pixels.show()
            self.usb_layout.write(ch)
            time.sleep(0.1)
        self.keyboard_state = old_state
        self.display_text("")
        self.usb_kbd.send(Keycode.ENTER)
        self.set_keyboard_state(old_state)

    def print_key(self, symbol, bits):
        # Print a key with its bits on the display
        print("Printing:", symbol, bits)
        self.send_animated_text_to_keyboard(symbol)

        if RIGHT_HANDED:
            top_line = "        "
            bottom_line = ""
            for bit in (1, 2, 4, 8, 16, 32):
                text = "[X] " if (bits & bit) != 0 else "[ ] "
                if bit < 4:
                    bottom_line += text
                else:
                    top_line += text
        else:
            top_line = ""
            bottom_line = "                "
            for bit in (32, 16, 8, 4, 2, 1):
                text = "[X] " if (bits & bit) != 0 else "[ ] "
                if bit < 4:
                    bottom_line += text
                else:
                    top_line += text
        self.send_animated_text_to_keyboard(top_line)
        self.send_animated_text_to_keyboard(bottom_line)
        self.send_animated_text_to_keyboard("-----------------------")

    def print_decode(self, title, decode_dict):
        # Print the decode dictionary on the display
        if self.help_proc.print_stop_check():
            return
        self.send_animated_text_to_keyboard(title)
        for item in sorted(decode_dict.items(), key=lambda x: x[1]):
            bits = item[0]
            symbol = item[1]
            self.print_key(symbol, bits)
            if self.help_proc.print_stop_check():
                return

    def print_control(self, title, decode_dict):
        # Print the control dictionary on the display
        if self.help_proc.print_stop_check():
            return
        self.send_animated_text_to_keyboard(title)
        for item in sorted(decode_dict.items(), key=lambda x: x[1][0]):
            bits = item[0]
            symbol = item[1][0]
            description = item[1][0] + " : " + item[1][2]
            self.print_key(description, bits)
            if self.help_proc.print_stop_check():
                return

    def build_reverse_lookup(self, key_lookup_dict):
        # Build a reverse lookup dictionary for characters
        result = {}
        for item in key_lookup_dict.items():
            bits = item[0]
            symbol = item[1]
            result[symbol] = bits
        return result

    def build_reverse_lookups(self):
        # Build reverse lookup dictionaries for text, numbers, and symbols
        self.text_reverse_lookup = self.build_reverse_lookup(self.text_decode)
        self.num_reverse_lookup = self.build_reverse_lookup(self.num_decode)
        self.sym_reverse_lookup = self.build_reverse_lookup(self.sym_decode)

    def lookup_character(self, ch):
        # Lookup the character in the reverse lookup dictionaries
        low_ch = ch.lower()
        if low_ch in self.text_reverse_lookup:
            bits = self.text_reverse_lookup[low_ch]
            state = PicoChord.LOWER_CASE_KEYS if low_ch == ch else PicoChord.UPPER_CASE_KEYS
            return (ch, bits, state)
        if ch in self.num_reverse_lookup:
            bits = self.num_reverse_lookup[ch]
            return (ch, bits, PicoChord.NUMBER_KEYS)
        if ch in self.sym_reverse_lookup:
            bits = self.sym_reverse_lookup[ch]
            return (ch, bits, PicoChord.SYMBOL_KEYS)
        return None

    def get_command(self, name, commands):
        # Get a command from the user
        self.send_text_to_keyboard(name)
        key_no = 0
        command_keys = []
        self.pixels.fill(Col.BLACK)
        for command in commands:
            col = command[0]
            text = command[1]
            self.send_text_to_keyboard(Col.names[col] + ' : ' + text)
            key = self.keys[key_no]
            self.pixels[key.pixel] = col
            key_no += 1
        self.pixels.show()
        self.wait_for_all_keys_up()
        while True:
            for pos in range(0, key_no):
                key = self.keys[pos]
                debounce = key.debounce
                debounce.update()
                if debounce.fell:
                    return commands[pos][0]

    def start_mode(self, mode):
        # Start a new mode
        self.mode = mode
        processor = self.mode_processors[self.mode]
        processor.start()

    def handle_text_control(self):
        # Handle text control command
        if self.mode == PicoChord.GAME_MODE:
            self.start_lower_case_text()
        else:
            self.start_mode(PicoChord.TEXT_MODE)

    def print_keys(self):
        # Print the keys on the display
        self.print_decode("Text", self.text_decode)
        self.print_decode("Numbers", self.num_decode)
        self.print_decode("Symbols", self.sym_decode)
        self.print_control("Control", self.command_actions)

    def __init__(self, i2c_sda, i2c_scl, pixel_pin, key_switches):
        # Initialize the PicoChord keyboard
        hello_message = "PICO Chord " + str(version)
        print(hello_message)
        self.key_down_col = Col.RED
        self.pixels = neopixel.NeoPixel(pixel_pin, 6, auto_write=False)
        self.pixels.fill(Col.BLUE)
        self.pixels.show()
        self.usb_kbd = Keyboard(usb_hid.devices)
        self.usb_layout = KeyboardLayoutUK(self.usb_kbd)
        self.i2c = busio.I2C(i2c_scl, i2c_sda)
        self.display = segments.Seg14x4(self.i2c)
        self.scroll_text(hello_message)
        self.keys = []
        for switch in key_switches:
            key = Key(keyboard=self, switch=switch)
            self.keys.append(key)

        self.text_decode = {
            12: 'a', 56: 'b', 10: 'c', 14: 'd', 4: 'e', 30: 'f', 48: 'g', 34: 'h',
            6: 'i', 50: 'j', 18: 'k', 38: 'l', 60: 'm', 24: 'n', 8: 'o', 62: 'p',
            40: 'q', 22: 'r', 16: 's', 20: 't', 32: 'u', 36: 'v', 54: 'w', 58: 'x',
            26: 'y', 42: 'z', 2: ' ', 52: ',', 28: '.'
        }

        self.num_decode = {
            8: '0', 2: '1', 6: '2', 14: '3', 30: '4', 62: '5', 32: '6', 48: '7',
            56: '8', 60: '9', 52: ',', 28: '.'
        }

        self.sym_decode = {
            2: ' ', 52: ',', 28: '.', 54: ':', 50: ';', 42: '%', 4: '=', 22: '&',
            14: '(', 56: ')', 16: '$', 12: '@', 62: '+', 34: '#',
            24: '-', 58: '!', 26: '?', 18: '/', 30: '{', 60: '}',
            10: '[', 40: ']', 36: '\\', 20: '*', 6: '<', 48: '>'
        }

        self.build_reverse_lookups()
        self.state_decodes = (self.text_decode, self.text_decode, self.num_decode, self.sym_decode)

        self.command_actions = {
            1: ("caps", lambda x: self.do_toggle_caps_lock(), "Toggle CAPS lock"),
            13: ("del", lambda x: self.usb_kbd.send(Keycode.BACKSPACE), "Backspace and delete"),
            17: ("sym", lambda x: self.set_keyboard_state(PicoChord.SYMBOL_KEYS), "Set symbol mode (magenta)"),
            21: ("text", lambda x: self.handle_text_control(), "Set text mode (blue)"),
            25: ("num", lambda x: self.start_number_text(), "Set numeric mode (green)"),
            29: ("for", lambda x: self.usb_kbd.send(Keycode.RIGHT_ARROW), "Move cursor right"),
            44: ("ret", lambda x: self.usb_kbd.send(Keycode.ENTER), "Enter key"),
            57: ("bak", lambda x: self.usb_kbd.send(Keycode.LEFT_ARROW), "Move cursor left"),
            33: ("Help", lambda x: self.start_mode(PicoChord.HELP_MODE), "Type help information"),
            49: ("Game", lambda x: self.start_mode(PicoChord.GAME_MODE), "Start the game")
        }

        test_texts = (
            "abcdefghijklmnopqrstuvwxyz",
            "the quick brown fox jumps over the lazy dog",
            "Jackdaws love my big sphinx of quartz.",
            "The five boxing wizards jump quickly.",
            "A Capital Idea",
            "1234567890",
            "I am 21 years old",
            "if a<b print(\"hello\")"
        )

        self.game_proc = GameProcessor(self, test_texts)
        self.text_proc = TextProcessor(self)
        self.help_proc = HelpProcessor(self)

        self.mode_processors = {
            PicoChord.TEXT_MODE: self.text_proc,
            PicoChord.HELP_MODE: self.help_proc,
            PicoChord.GAME_MODE: self.game_proc
        }

        self.wait_for_all_keys_up()
        self.character_bits = 0
        self.assembling_bits = False
        self.mode = PicoChord.TEXT_MODE
        self.start_mode(PicoChord.TEXT_MODE)

        self.pixels.show()

    def key_down(self):
        # Check if any key is currently pressed
        for key in self.keys:
            debounce = key.debounce
            debounce.update()
            if not debounce.value:
                return True
        return False

    def wait_for_all_keys_up(self):
        # Wait until all keys are released
        while True:
            if self.key_down():
                continue
            return

    def test(self):
        # Test method to display key presses on the segment display
        self.display.print("Test")
        while True:
            for key in self.keys:
                debounce = key.debounce
                bit = key.bit
                debounce.update()
                if debounce.fell:
                    print("Key down:", bit)
                    self.pixels[key.pixel] = Col.RED
                    self.display.fill(0)
                    self.display.print(f"D{bit}")
                if debounce.rose:
                    print("Key up:", bit)
                    self.pixels[key.pixel] = Col.GREY
                    self.display.fill(0)
                    self.display.print(f"U{bit}")
            self.pixels.show()

    def update_keys(self):
        # Update the state of all keys
        for key in self.keys:
            key.update()

    def update(self):
        # Update the keyboard state
        self.update_keys()
        processor = self.mode_processors[self.mode]
        processor.update()
        self.pixels.show()

    def run(self):
        # Main run loop for the keyboard
        while True:
            self.update()

if RIGHT_HANDED:
    key_switches = [
        Switch(pin=board.GP15, pixel=0, bit=1),  # control
        Switch(pin=board.GP14, pixel=1, bit=2),  # thumb
        Switch(pin=board.GP13, pixel=2, bit=4),  # index
        Switch(pin=board.GP12, pixel=3, bit=8),  # middle
        Switch(pin=board.GP11, pixel=4, bit=16), # ring
        Switch(pin=board.GP10, pixel=5, bit=32)  # little
    ]
else:
    key_switches = [
        Switch(pin=board.GP15, pixel=0, bit=32),  # control
        Switch(pin=board.GP14, pixel=1, bit=16),  # thumb
        Switch(pin=board.GP13, pixel=2, bit=8),  # index
        Switch(pin=board.GP12, pixel=3, bit=4),  # middle
        Switch(pin=board.GP11, pixel=4, bit=2), # ring
        Switch(pin=board.GP10, pixel=5, bit=1)  # little
    ]

keyboard = PicoChord(i2c_sda=board.GP0, i2c_scl=board.GP1,
            pixel_pin=board.GP17,
            key_switches=key_switches)
keyboard.run()


```