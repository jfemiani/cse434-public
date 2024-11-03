### Illustrative Texture Synthesis Tutorial

In this tutorial, we'll explore an illustrative approach to texture synthesis. This approach draws inspiration from general texture synthesis techniques, including ideas from Wei and Levoy, as well as Efros and Leung, but is not meant to faithfully recreate any specific existing method. Instead, it serves to demonstrate key concepts in texture synthesis by matching neighborhoods to generate visually plausible textures. This method allows us to generate visually plausible textures by finding and expanding local patterns present in the source image.

#### Overview

The Wei-Levoy method for texture synthesis is an example of pixel-based synthesis, where each pixel of the new texture is generated based on the information gathered from the input texture. The core idea revolves around using a non-parametric sampling strategy to extend the texture by selecting pixels that are statistically similar to existing neighborhoods. The similarity criterion draws inspiration from n-gram modeling, where we attempt to predict the next element in a sequence based on a preceding context.

#### Step-by-Step Implementation

##### 1. **Input Texture and Initialization**

- Start with an example texture, typically an image with a repeating pattern.
- Use images from the MIT Vision Texture (VisTex) database, which can be downloaded from [VisTex](http://vismod.media.mit.edu/pub/VisTex/VisTex.tar.gz). The dataset contains PBM/PPM images that are well-suited for texture synthesis experiments.
- Create an output canvas that is initialized with a random seed value or copied directly from a small patch of the input image. This canvas will be iteratively filled.

##### 2. **Neighborhood Matching**

- For every pixel to be synthesized, the core idea is to determine its value based on existing neighboring pixels.
- Define a neighborhood window (e.g., a 5x5 or 7x7 patch) around the pixel you want to synthesize. This neighborhood serves as the context, akin to an n-gram in text generation.
- Only consider the causal neighborhood: Flatten the neighborhood and use only the pixels that have already been synthesized (i.e., the first half of the neighborhood). This ensures that you are only using pixels that are already known, which makes the synthesis process more efficient and avoids ambiguity.
- Search within the original texture for pixels that have a similar causal neighborhood as the context defined in the output canvas.

##### 3. **Finding the Best Match**

- Calculate a distance metric, typically the sum of squared differences (SSD), between the causal neighborhood of the pixel to be synthesized and neighborhoods in the input texture.
- Keep track of the sampled locations in a separate array called `sample_locations`. Each time you sample a location from the input texture, store its coordinates in this array.
- When finding the best match for a new pixel, use a refined search strategy:
  1. Take as input the `input_texture`, the `causal_neighborhood`, the `sample_locations`, and the row (`r`) and column (`c`) of the current output pixel.
  2. Search in a 3x3 window around the `(r, c)` location to determine possible match locations from the input texture.
  3. Append a configurable number of additional random locations (e.g., 1000).
  4. Test all these locations to find the one with the minimum SSD compared to the causal neighborhood.

##### 4. **Synthesis Process**

- Proceed pixel-by-pixel, updating the output canvas with the chosen pixel values.
- The order of synthesis is crucial: Wei and Levoy's approach alternates the scanning order. For even-numbered iterations, scan from top to bottom, while for odd-numbered iterations, scan from bottom to top. This alternating scan pattern helps in spreading the synthesized information evenly across the output canvas.
- At the end of each iteration, flip the input and reference textures by reversing them vertically. This swapping ensures that the newly generated texture serves as the reference for the next iteration, leading to more natural blending of features.
- Continue until the entire output canvas is filled.

##### 5. **Optimization Considerations**

- **Coherence**: Wei and Levoy use a clever optimization to speed up synthesis by leveraging spatial coherence. If a nearby pixel has already been synthesized, it's likely that a similar match will work well for the current pixel, reducing the need to search the entire input image.
- **Multi-Resolution**: For complex textures, consider synthesizing at multiple resolutions. Start by synthesizing a low-resolution version and progressively add details at higher resolutions. This is similar to coarse-to-fine n-gram modeling, where you refine predictions based on more detailed contexts.

#### Example Python Implementation Using VisTex Images

##### Setup

```python
import numpy as np
from skimage import io
import random
import matplotlib.pyplot as plt

# Load the input texture from VisTex (e.g., PPM format)
input_texture = io.imread('https://vismod.media.mit.edu/vismod/imagery/VisionTexture/Images/Reference/Flowers/Flowers.0003.ppm')
height, width, channels = input_texture.shape

# Define output dimensions
output_height, output_width = 200, 200  # Example dimensions

# Initialize output texture with random values
output_texture = np.random.randint(0, 256, (output_height, output_width, channels), dtype=np.uint8)

# Track sampled locations
sample_locations = np.zeros((output_height, output_width, 2), dtype=int)
# Initialize sample locations with random row and column values constrained by neighborhood size
sample_locations[..., 0] = np.random.randint(0, height - 5, (output_height, output_width))
sample_locations[..., 1] = np.random.randint(0, width - 5, (output_height, output_width))

# Visualization function
def visualize_textures(input_texture, output_texture, sample_locations):
    """Visualize input texture, output texture, and sample locations."""
    fig, ax = plt.subplots(1, 3, figsize=(18, 6))
    ax[0].imshow(input_texture)
    ax[0].set_title('Input Texture')
    ax[1].imshow(output_texture)
    ax[1].set_title('Output Texture')
    ax[2].imshow(sample_locations[..., 0], cmap='viridis')
    ax[2].set_title('Sample Locations (Row)')
    plt.show()

# Initial visualization
visualize_textures(input_texture, output_texture, sample_locations)
```



##### Search Function

The following function, `find_best_match`, plays a crucial role in synthesizing textures based on an existing reference image. The function searches for the best matching neighborhood from the input texture to use for synthesizing each new pixel in the output texture. It aims to find a pixel that most closely matches the given causal neighborhood (already synthesized pixels around the target pixel), ensuring that the generated output is visually coherent. The function employs a combination of a local search (around the current pixel) and a broader search with random samples to achieve this.

```python
def find_best_match(input_texture, causal_neighborhood, sample_locations, r, c, search_radius=5, num_random_samples=1000):
    # Start by collecting potential match locations
    possible_locations = []
    
    # Search in a 3x3 window around the current (r, c) location
    for dr in range(-1, 2):
        for dc in range(-1, 2):
            nr, nc = r + dr, c + dc
            if 0 <= nr < input_texture.shape[0] - 5 and 0 <= nc < input_texture.shape[1] - 5:
                possible_locations.append((nr, nc))
    
    # Append additional random locations
    for _ in range(num_random_samples):
        nr = random.randint(0, input_texture.shape[0] - 5 - 1)
        nc = random.randint(0, input_texture.shape[1] - 5 - 1)
        possible_locations.append((nr, nc))
    
    # Find the best match among all possible locations
    best_match = None
    min_ssd = float('inf')
    match_location = None
    
    for (nr, nc) in possible_locations:
        candidate_neighborhood = input_texture[nr:nr+5, nc:nc+5].flatten()
        ssd = np.sum((causal_neighborhood - candidate_neighborhood[:len(causal_neighborhood)])**2)
        if ssd < min_ssd:
            min_ssd = ssd
            best_match = input_texture[nr, nc]
            match_location = (nr, nc)
    
    return best_match, match_location
```



#####

##### Main Synthesis Loop

The main synthesis loop iteratively fills in the output texture by finding the best match for each pixel from the input texture. It processes each pixel by extracting its causal neighborhood, finding a matching patch from the input, and updating the output accordingly. The input and output are flipped between iterations to enhance the blending of features across the texture.

```python
# Define the neighborhood size (e.g., 5x5)
neighborhood_size = 5
flipped = False
for iteration in range(output_height * output_width):
    
    for y in range(neighborhood_size, output_height - neighborhood_size):
        for x in range(neighborhood_size, output_width - neighborhood_size):
            # Extract the causal neighborhood from the output texture
            neighborhood = output_texture[y-neighborhood_size:y+1, x-neighborhood_size:x+1].flatten()
            causal_neighborhood = neighborhood[:len(neighborhood) // 2]
            
            # Find the best match in the input texture
            best_match, match_location = find_best_match(input_texture, causal_neighborhood, sample_locations, y, x)
            
            # Update the output texture with the matched pixel
            output_texture[y, x] = best_match
            
            # Keep track of the matched location
            sample_locations[y, x] = match_location
    
    # Flip the input and output textures for the next iteration
    input_texture = input_texture[::-1]
    output_texture = output_texture[::-1]
    flipped = not flipped

if flipped:
    input_texture = input_texture[::-1]
    output_texture = output_texture[::-1]

# Save or display the output texture
io.imsave('synthesized_texture.png', output_texture)
```

##### Key Insights and Limitations

- **Local Context**: The Wei-Levoy method excels at capturing local structures due to its n-gram-like use of neighborhoods. However, it may struggle with global consistency, especially for textures with long-range dependencies.
- **Computational Complexity**: Searching for the best match at each pixel can be computationally intensive. The use of spatial coherence, as proposed by Wei and Levoy, helps mitigate this cost.
- **Multi-Resolution Synthesis**: For more challenging textures, multi-resolution synthesis can help capture both coarse and fine details effectively.

#### Conclusion

The Wei-Levoy method for texture synthesis provides a powerful way to generate plausible textures based on an example, building on ideas similar to n-grams by leveraging local context to predict new values. This pixel-based approach is versatile, with applications ranging from graphics to procedural content generation. By following the steps outlined above and using VisTex images, you should be able to implement a basic version of this algorithm and experiment with synthesizing various textures.

