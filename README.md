# Filters
An Arduino finite impulse response and infinite impulse response filter library.

## Digital filters
This library implements digital [finite impulse response](https://en.wikipedia.org/wiki/Finite_impulse_response) filters (FIR) 
and [infinite impulse response](https://en.wikipedia.org/wiki/Infinite_impulse_response) filters (IIR). 
32-bit floating point arithmetic is used.

The difference equation for FIR filters is given by  
![FIR difference equation](https://wikimedia.org/api/rest_v1/media/math/render/svg/c43ba6c329a471401e87fe17c6130d801602ffdf)  
To initialize it in code, you can use:
```cpp
const float b_coefficients[] = { b_0, b_1, b_2, ... , b_N };
FIRFilter fir(b_coefficients);
```
The difference equation for FIR filters is given by  
![IIR difference equation](https://wikimedia.org/api/rest_v1/media/math/render/svg/bddf0360f955643eeedc46d9be4b8f2d4f4d288f)  
```cpp
const float b_coefficients[] = { b_0, b_1, b_2, ... , b_P };
const float a_coefficients[] = { a_0, a_1, a_2, ... , a_Q };
IIRFilter iir(b_coefficients, a_coefficients);
```

## Usage
The `filter` method takes the new (raw) value as an intput, and outputs the new (filtered) value.

```cpp
float raw_value = getRawValue();
float filtered_value = fir.filter(raw_value);
```
You can use multiple filters on the input.
```cpp
float filtered_value = fir.filter(iir.filter(raw_value));
```

## Implementation
This library uses the straightforward [Direct Form I](https://en.wikipedia.org/wiki/Digital_filter#Direct_form_I) implementation.  
This is basically just calculating the difference equation in the form expressed above.  
In this documentation, we'll only look at the implementation of the FIR filter, but the IIR filter implementation is very similar.  
![FIR difference equation](https://wikimedia.org/api/rest_v1/media/math/render/svg/c43ba6c329a471401e87fe17c6130d801602ffdf)  
All we have to do is multiplying the previous input samples with their respective coefficients. 

![implementation-1](https://raw.githubusercontent.com/tttapa/Filters/explain-implementation/Filters-1.png)
![implementation-2](https://raw.githubusercontent.com/tttapa/Filters/explain-implementation/Filters-2.png)

### Code
Let's start with the easiest part: calculating the dot product of x and b_j.
```cpp
float sum = 0;
for (uint8_t i = 0; i < 4; i++) {
  sum += x[i] * b_j[i];
}
```
`sum` now contains the filter's output value.  

To calculate the extended b vector `coeff_b = { b_N, ..., b_2, b_1, b_0, b_N, ..., b_2, b_1 }` from a given vector `b = { b_0, b_1, b_2, ..., b_N }`, the following loop is used:
```cpp
coeff_b = new float[2*N-1];
for (uint8_t i = 0; i < 2*N-1; i++) {
  coeff_b[i] = b[(2*N - 1 - i)%N];
} 
```
To find the correct b_j vector, we can use the following:
```cpp
float *b_j = &coeff_b[N - 1 - j];
```
To update the x ring buffer with a new sample, we have to insert it at position j:
```cpp
x[j] = newSample;
```
Finally, we'll need some logic to increment and wrap around index j:
```cpp
j++;
if(j == N)
  j = 0;
```
