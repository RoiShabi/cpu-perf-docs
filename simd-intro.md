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