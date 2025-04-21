# cpu-perf-docs
**Read Previous: [Atomic Compare Exchange](./atomic-compare-exchange.md)**

## simd table content

- What is simd
- SWAR
- When to, When not to
- The raise of GPU
- compare vendor support avx512 vs apple vs amd vs arm&risc

## SIMD

SIMD, stands for Single Instruction Multiple Data, is a multiprocessing methodology. As the name imply, SIMD is the operation of one instruction over multiple elements, at the same time (real hardware parallelism). It is used massively in signal processing, image processing, audio processing, physics simulation, etc.

#### SIMD Conceptual Demonstration

The following application runs grayscale transformation over an image (array of pixels).
```C++
struct rgb
{
    uint8_t red, green, blue;
}

inline void convert_pixel_rgb_to_gray(rgb* colored_pixel, uint8_t* dest_gray_pixel)
{
    uint16_t sum = colored_pixel->red + colored_pixel->green + colored_pixel->blue;
    uint16_t color_average = sum/3;
    *dest_gray_pixel = static_cast<uint8_t>(color_average);
};

int main()
{
    constexpr size_t PIXEL_COUNT = 2024;
    rgb colored_pixels[PIXEL_COUNT];
    uint8_t gray_pixels[PIXEL_COUNT];
    fill_pixels(colored_pixels); // it should write data to our array somehow

    for (size_t i = 0; i < PIXEL_COUNT; i++)
    {
        convert_pixel_rgb_to_gray(&colored_pixels[i], &gray_pixels[i]);
    }

    return 0;
}
```

SIMD approach, utilizes special instruction-set to process N element on the same iteration:
```C++
struct rgb
{
    uint8_t red, green, blue;
}

inline void convert_4pixels_rgb_to_gray(rgb* colored_4pixels, uint8_t* dest_gray_4pixels)
{
    // implementation is out of scope currently
};

int main()
{
    constexpr size_t PIXEL_COUNT = 2024;
    rgb colored_pixels[PIXEL_COUNT];
    uint8_t gray_pixels[PIXEL_COUNT];
    fill_pixel(colored_pixels); // it should write data to our array somehow

    for (size_t i = 0; i < PIXEL_COUNT; i+=4)
    {
        convert_4pixels_rgb_to_gray(&colored_pixels[i], &gray_pixels[i]);
    }

    return 0;
}
```

For an environment where the processor's ALU is the bottleneck, the SIMD approach can decrease computation time by up to 4 times. We should say here that the performance improvement is never as good as the theoretical improve, because we do have memory fetch overhead, and real world scenarios tend to struggle against the method's limitations.

### Array Processing

Array processing is a subset of SIMD, where each operation cover multiple elements (commonly known as vector), similar to the previous example. It uses dedicated vector registers and vector instructions to perform operations on multiple data elements simultaneously (128 bits, 256 and even 512).

### SWAT

SIMD Within A Register, is another subset of SIMD, where regular instructions are used to run an operation over multiple elements at the same time. Small sized elements are packed into a single cpu register, and with some tricks an operation runs on all the elements.

#### Example - String Lookup

In this example we are going to iterate a string (char*) until we find the null terminator (0x00 byte). Lets start with the trivial solution.
```C
size_t search_end_of_string(char* input)
{
    size_t index = 0;
    for(; input[index] != '\0'; index++)
    {}

    return index;
}
```

For an input like ***Linux Kernel Development 3rd Edition*** book, the characters count is 995,022 (according to Foxit PDF Reader). Calculating it with the current loop takes 995,022 iterations. If we run smart swap algorithm we can benefit massively by reducing the loops count. Lets write an example:
```C
typedef uint64_t aliasing_uint64_t __attribute__((__may_alias__));

size_t search_end_of_string_swar(const char *input) {
    const char *start = input;

    // Phase 1: byte-by-byte until 8-byte aligned
    while ((uintptr_t)input % 8 != 0) { //uintptr_t cast is for doing aritmentics on pointer type
        if (*input == '\0') {
            return input - start;
        }
        ++input;
    }

    // Phase 2: 64-bit SWAR scan
    while (true) {
        uint64_t chunk = *(aliasing_uint64_t*)input; // Safe type cast (we avoid compiler optimizations which may harm functionality)

        // This is tricky code, explanation exists below the code block
        uint64_t has_zero = (chunk - 0x0101010101010101ULL) &
                            ~chunk &
                            0x8080808080808080ULL;

        if (has_zero != 0) {
            // Fallback loop for the 8-bytes containing the zero
            for (int i = 0; i < 8; ++i) {
                if (input[i] == '\0') {
                    return (input - start) + i;
                }
            }
        }

        input += 8;
    }
}
```

This loop runs 124,378 iterations, with additional 6 iterations of the last loop, and 0-7 more iterations of the first loop at unaligned input. For some benchmarks I ran the results were 7.8 times faster (measuring time).
> Good to know: this method also have vectorized (classical SIMD) implementation, and for 256bit register the improvement was 27.9 times faster.

##### Explain the lookup hack