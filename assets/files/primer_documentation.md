# Primer Image Processing Tool Documentation

## Overview

This is a comprehensive image processing application built with Gradio that provides advanced background removal, image positioning, and classification capabilities. The tool is designed for batch processing of product images with various canvas sizes, background removal methods, and intelligent positioning algorithms.

## Key Features

### 🎯 Core Functionality
- **Multiple Background Removal Methods**: BiRefNet, BRIA, RemBG, PhotoRoom API, and more
- **Intelligent Image Positioning**: Advanced algorithms for optimal product placement
- **Batch Processing**: Handle multiple images, ZIP files, and RAW files (.arw)
- **Canvas Size Presets**: 80+ predefined canvas sizes for different brands and platforms
- **Auto-Classification**: AI-powered image classification using Qwen 2.5VL model
- **Smart Snap Detection**: Automatic edge detection for optimal positioning

### 🖼️ Supported Formats
- **Input**: PNG, JPG, JPEG, BMP, GIF, WebP, TIFF, AVIF, ARW (Sony RAW), ZIP, RAR
- **Output**: PNG, JPG with color profile preservation
- **Special Features**: CMYK color space handling, ICC profile preservation

## Architecture

### Main Components

#### 1. Background Removal Engines
```python
# Available methods:
- 'rembg': General-purpose background removal
- 'bria': BRIA AI model for precise segmentation
- 'photoroom': PhotoRoom API integration
- 'birefnet': BiRefNet model for high-quality results
- 'birefnet_2': Alternative BiRefNet configuration
- 'birefnet_hr': High-resolution BiRefNet processing
- 'none': No background removal
- 'none_with_padding': Custom positioning without BG removal
```

#### 2. Image Classification System
Uses Qwen 2.5VL vision model for intelligent product classification:
- Automatic object detection and categorization
- CSV/Excel sheet integration for classification rules
- Dynamic padding adjustment based on classification
- Person detection for special positioning rules

#### 3. Positioning Algorithms

##### Auto Snap Detection
```python
def auto_detect_zero_padding(image_path, bg_method='bria', threshold=30, 
                           edge_threshold=0.10, distance_threshold=0.15)
```
- **Method 1**: Advanced object boundary analysis using OpenCV
- **Method 2**: Color-based analysis for fallback detection
- **Method 3**: Background removal confirmation
- **Smart Thresholds**: Dynamic thresholds based on image dimensions

##### Position Logic
- **Cropped Edge Detection**: Identifies which sides contain cropped objects
- **Aspect Ratio Preservation**: Maintains original proportions
- **Canvas Fitting**: Intelligent scaling and positioning
- **Snap-to-Edge**: Zero padding for specific sides

### 4. Canvas Size Presets

#### Lifestyle/Social Media (L/S) - 1080x1080
```
Aetrex-L/S, Allbirds-L/S, Backjoy-L/S, Beecho-L/S, Billabong-L/S,
Birkenstock-L/S, Bratpack-L/S, Columbia-L/S, DC-L/S, Delsey-L/S,
[... 40+ more brands]
```

#### E-commerce Platforms
- **Zalora**: 762x1100 format for marketplace listings
- **DOTCOM**: Various sizes for brand websites (1000x1000, 1124x1285, etc.)

## Advanced Features

### 1. RAW File Support
```python
def convert_arw_to_pil(arw_path):
    """Convert Sony RAW (.arw) files to PIL Image objects"""
```
- Automatic RAW file detection and conversion
- Support for Sony ARW format
- Fallback handling for unsupported RAW formats

### 2. Color Profile Management
```python
def convert_image_to_rgb(img):
    """Consistently convert images while preserving color accuracy"""
```
- CMYK to RGB conversion with ICC profile preservation
- Automatic color space detection
- Profile-aware image processing

### 3. Text Detection and Auto-Cropping
```python
def detect_reference_text(image_path, target_text="*This image is for size reference only"):
    """Detect reference text using EasyOCR and auto-crop if found"""
```
- OCR-based text detection using EasyOCR
- Automatic bottom cropping for reference text removal
- Configurable crop dimensions

### 4. Smart Positioning Logic

#### Zero Padding System
- **Manual Snap Controls**: Individual edge snapping
- **Auto Snap Detection**: AI-powered edge detection
- **Constraint-Based Sizing**: Width/height constraints based on padding

#### Position Algorithms
1. **position_logic_old()**: Legacy positioning with cropped edge detection
2. **position_with_padding()**: Fill-width positioning without whitespace
3. **Auto-detection**: Intelligent snap point identification

## API Integration

### Qwen 2.5VL Classification
```python
def inference_with_api(image_path, prompt, sys_prompt, model_id="qwen2.5-vl-72b-instruct"):
    """Classify images using Qwen vision model"""
```
- Base64 image encoding
- Retry logic for connection errors
- Configurable model parameters

### PhotoRoom API
```python
def remove_background_photoroom(input_path):
    """Professional background removal via PhotoRoom API"""
```
- API key authentication
- High-quality segmentation results
- Error handling and fallbacks

## User Interface

### Gradio Interface Components

#### Input Controls
- **File Uploads**: Multiple input types (main, special rotation, no-BG)
- **Background Method**: Radio selection for BG removal
- **Canvas Size**: Dropdown with 80+ presets
- **Output Format**: PNG/JPG selection
- **Classification**: Vision model toggle

#### Processing Controls
- **Snap Controls**: Individual edge snapping checkboxes
- **Auto Snap**: Intelligent detection toggle
- **Rotation/Flip**: Transformation options
- **Workers**: Parallel processing configuration

#### Output Display
- **Gallery**: Processed image preview with captions
- **Comparison View**: Side-by-side original vs processed
- **Classification Results**: Detailed processing information
- **Download**: ZIP file with all processed images

## Processing Workflow

### 1. Input Processing
```
Input Files → Format Detection → RAW Conversion (if needed) → Text Detection → Auto-Crop
```

### 2. Classification (Optional)
```
Image → Qwen 2.5VL → Classification → Sheet Lookup → Padding Override
```

### 3. Background Removal
```
Image → Selected Method → Mask Generation → Color Fidelity Preservation
```

### 4. Positioning
```
Mask → Bounding Box → Cropped Edge Detection → Position Calculation → Canvas Placement
```

### 5. Post-Processing
```
Canvas → Rotation/Flip → Watermark → Twibbon → Color Profile → Output
```

## Configuration Examples

### Basic Processing
```python
process_single_image(
    image_path="product.jpg",
    output_folder="output/",
    bg_method="bria",
    canvas_size_name="Allbirds-L/S",
    output_format="JPG",
    bg_choice="white"
)
```

### Advanced with Classification
```python
process_single_image(
    image_path="product.jpg",
    output_folder="output/",
    bg_method="birefnet",
    canvas_size_name="Columbia-DOTCOM",
    use_qwen=True,
    sheet_data=classification_df,
    auto_snap=True
)
```

### Batch Processing
```python
process_images(
    input_files=["img1.jpg", "img2.png"],
    bg_method="photoroom",
    num_workers=4,
    zero_padding=["bottom", "left"],
    auto_detect_padding=True
)
```

## Performance Optimization

### Parallel Processing
- ThreadPoolExecutor for concurrent image processing
- Configurable worker count (1-16)
- Memory management for large batches

### Model Caching
- LRU cache for BiRefNet models
- Singleton pattern for EasyOCR reader
- GPU memory management

### Memory Efficiency
- Temporary file cleanup
- Progressive processing for large datasets
- Garbage collection for model inference

## Error Handling

### Robust Fallbacks
- Multiple background removal method fallbacks
- Color space conversion error handling
- File format compatibility checks
- Network timeout handling for API calls

### Logging and Debugging
- Comprehensive processing logs
- Color value debugging (disabled in production)
- Classification result tracking
- Performance timing

## Dependencies

### Core Libraries
```python
PIL (Pillow)           # Image processing
gradio                 # Web interface
numpy                  # Numerical operations
opencv-python          # Computer vision
torch                  # Deep learning
transformers           # AI models
```

### Optional Dependencies
```python
rawpy                  # RAW file support
easyocr                # Text detection
rembg                  # Background removal
pandas                 # Data processing
requests               # API calls
```

## Usage Examples

### 1. E-commerce Product Processing
```python
# Process product images for online store
results = process_images(
    input_files="products.zip",
    bg_method="photoroom",
    canvas_size="Allbirds-DOTCOM",
    bg_choice="white",
    output_format="JPG"
)
```

### 2. Social Media Content
```python
# Create social media ready images
results = process_images(
    input_files=lifestyle_images,
    bg_method="birefnet",
    canvas_size="Columbia-L/S",
    bg_choice="transparent",
    snap_to_bottom=True
)
```

### 3. Automated Classification Pipeline
```python
# AI-powered classification and processing
results = process_images(
    input_files=mixed_products,
    bg_method="bria",
    use_qwen=True,
    sheet_data=classification_rules,
    auto_snap=True,
    num_workers=8
)
```

## Best Practices

### 1. Input Preparation
- Use high-resolution source images
- Ensure consistent lighting
- Remove obvious background elements manually if needed

### 2. Method Selection
- **BiRefNet**: Best quality, slower processing
- **BRIA**: Good balance of speed and quality
- **PhotoRoom**: Professional results, requires API key
- **RemBG**: Fast, general-purpose

### 3. Canvas Size Selection
- Match target platform requirements
- Consider aspect ratios of source images
- Use brand-specific presets when available

### 4. Performance Tuning
- Adjust worker count based on system resources
- Use appropriate background removal method for batch size
- Enable auto-snap for consistent results

## Troubleshooting

### Common Issues

#### 1. Memory Errors
- Reduce number of workers
- Process smaller batches
- Use CPU-only mode for BiRefNet

#### 2. Color Issues
- Check ICC profile preservation
- Verify CMYK conversion settings
- Use appropriate output format

#### 3. Positioning Problems
- Review cropped edge detection
- Adjust snap settings manually
- Check canvas size compatibility

#### 4. Classification Errors
- Verify sheet data format
- Check API key configuration
- Review classification column names

## Future Enhancements

### Planned Features
- Additional background removal methods
- More canvas size presets
- Enhanced classification models
- Real-time preview capabilities
- Batch processing optimization

### API Improvements
- RESTful API endpoint
- Webhook support for automated workflows
- Cloud storage integration
- Advanced analytics and reporting

---

## License and Credits

This tool integrates multiple open-source libraries and commercial APIs:
- BiRefNet: Advanced segmentation model
- BRIA: AI-powered background removal
- PhotoRoom: Professional API service
- Qwen 2.5VL: Vision-language model by Alibaba
- EasyOCR: Text detection library

For commercial use, ensure compliance with all integrated service terms and licensing requirements.
