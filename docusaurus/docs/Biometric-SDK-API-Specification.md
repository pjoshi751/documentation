# Biometrics SDK API Specifications
This document defines the interface for the Java Library providing the functional support for processing biometrics.

API specification version:  **0.9**

Published Date: June 11, 2020

## Revision Note
This is the first formal publication of the interface as a versioned specification. Earlier draft are superseded by this document. The interface is revamped to make it friendlier to programmers and also has a new method for conversion.

# Introduction
Mosip as a platform does not have any inbuilt capabilities to handle biometrics. It relies on external components and subsystems to perform all activities pertaining to biometrics. As a platform it defines formats, standards and interfaces for these external components and subsystems. The Biometrics SDK is a critical external component used for performing operations with biometric data in multiple mosip modules - Registration Client, Authentication and Registration Processor.


# SDK Initialization
## Purpose
The SDK initialization methods serves the dual purpose of sharing information about the SDK and performing any one time activities including initialization of internal variables and algorithms.
## Signature
`SDKInfo init(Map<String, String> initParams)`

**Input Parameters**
* initParams - Optional initialization parameters. Mosip can be setup to pass specific input parameters to the SDK. This could potentially include license keys, other flags.

**Output Parameters**
* SDKInfo - SDK information containing information on the provider, the api version supported and the list of supported biometric types and biometric functions supported within each biometric type.

**Errors/Exceptions**
* Initialization error
* License error
* InitParams error

## Implementation notes
The SDK can perform one time initialization activities such as license checks, initializing templates, caches and loggers. Initialization parameters can be configured in mosip and passed to the SDK during the Init call. These are typically Key-Value Pairs.

# Quality Check
## Purpose
The quality check method is used to determine if the biometrics data is of sufficient quality for uniqueness check and authentication matches.
## Signature
`Response<QualityCheck> checkQuality(BiometricRecord sample, List<BiometricType> modalitiesToCheck, Map<String, String> flags)`

**Input Parameters**
* sample - Biometric Record of a person containing a list of biometric segments in the BIR format. The segments could correspond to multiple modalities and could be FIR, IIR, or Face Image Record.
* modalitiesToCheck - List of biometric types to perform the quality check on. If this is null or empty all biometric types that the SDK supports can be checked. If present the modalities specified alone should be processed and the others present in the sample should be ignored.
* flags - An optional list of control flags as name value pairs that can be used to configure the behavior of the library.

**Output Parameters**
* QualityCheck object with a list of quality scores by modality. The quality score is on a scale of 100 - Higher is better.

**Errors/Exceptions**
* Unsupported biometric type
* Unsupported image format
* Processing error

## Implementation notes
The input biometric record has segments from multiple biometric types. The method is expected to process the segments of each biometric type together and return a single quality score for the biometric type. For example, if the record has the following segments - right iris, left iris, right thumb, left thumb - the method should return a quality score for biometric types iris and fingerprint. The iris quality score will factor the left iris and right iris segments, while the fingerprint quality will factor the left thumb and right thumb.
* The flags are optional by default.  


## Behavior per biometric type

**Fingerprint Segments**
* The biometric image record is a Fingerprint Image Record. The FIR structure is explained in a later section
* The image is a jpeg2000 format lossless image
* The quality score will be using NFIQ2 for 500 dpi images and NFIQ for other densities
* The analytics data returned can have information on finger index, liveness, etc.

**Iris Segments**
* The biometric image record is a Iris Image Record. The IIR structure is explained in a later section
* The image is a jpeg2000 format lossless image
* The quality score will be on a scale of 100 and will factor focus, blur, eyelid position etc.
* The analytics data returned can have information on eye index, eyelid position, iris obscuration, gaze angle etc.

**Face Segments**
* The biometric image record is a Face Image Record. The FaceIR structure is explained in a later section
* The image is a jpeg2000 format lossless image
* The quality score will be on a scale of 100 and will factor ICAO standards
* The analytics data returned can have information on tilt, missing landmarks, lighting etc.
{% hint style="info" %}
For new biometrics that come up the quality check could be different.
{% endhint %}

# Matcher
## Purpose
The matcher is used to perform checks if the provided input biometrics belong to the same person. The matcher has several use cases within mosip. It is used for 1:1 matches where a person's identity is verified using a biometrics match. It is also used for 1:n(few) matches for uniqueness checks.

## Signature
`Response<MatchDecision[]> match(BiometricRecord sample, BiometricRecord[] gallery, List<BiometricType> modalitiesToMatch, Map<String, String> flags)`

**Input Parameters**
* sample - Biometric Record of a person containing a list of biometric segments in the BIR format. The segments could correspond to multiple modalities and could be FIR, IIR, or Face Image Record.
* gallery - List of biometric records of people to match against. Each biometric record in this list will contain multiple segments that can be used in the match process. The smaller the gallery size the better the performance. This list will have one entry in a 1:1 match case and multiple entries in a 1:n(few) case.
* modalitiesToMatch - List of biometric types to perform the match on. If this is null or empty all biometric types that the SDK supports should be matched. If present the modalities specified alone should be processed and the others present in the sample can be ignored.
* flags - An optional list of control flags as name value pairs that can be used to configure the behavior of the library.

**Output Parameters**
* MatchDecision[] - Array of MatchDecision of the size same as the input parameter gallery. Each MatchDecision object will contain a list of decisions by biometric type. The decision information is a yes/no/error decision along with an analytics object which contain key value pairs that provide more insight into the decision.

**Errors/Exceptions**
* Unsupported biometric type
* Unsupported image format
* Missing segments in biometric record
* Processing error
{% hint style="info" %}
One of the example for the behaviour is threshold for match using which the SDK can take a decision.
{% endhint %}

## Behavior per biometric type
**Fingerprint**
* The biometric segment will have a jpeg2000 format lossless image or minutiae in an ISO template (FMR).
* The on record biometrics will be jpeg200 format lossless images or biometric extracts tagged to a specific extractor.
* Best matches are provided for an image to image match. The sample and on records data are both images. The matcher uses its own extraction algorithm on the images to be compared.
* In case the sample is not an image but minutiae (FMR) then the match accuracy and efficacy might be lower as the extraction templates and algorithms might not be the same
* In case the on record extract is a format that the matcher does not understand the match will not happen. Also if the comparison is between two extracts (Minutiae) the False Rejection and False Acceptance cases might be high.
* All fingers provided in the input are used for match. If the input finger segment is not identified as a particular finger, it is mathched against all fingers in the gallery record.
* The match decision will return a yes/no and optional anaytics with details such a confidence level.

**Iris**
* Input and on record images are jpeg2000 format lossless images or matchers own extract as provided by the extarctor API
* The matcher uses its own algorithm to perform extraction, segmentation and identifying patterns on the images being compared.
* All iris segments passed are used in match. In case the segment is not identified as left or right iris, it is matched against both irises in the gallery record.
* The match decision will return a yes/no and optional anaytics with details such a confidence level.

**Face**
* Input and on record images are jpeg2000 format lossless images or matchers own extract as provided by the extarctor API
* The matcher uses its own algorithm to perform extraction, segmentation and identifying landmarks on the images being compared.
* The match decision will return a yes/no and optional anaytics with details such a confidence level.

## Implementation notes
The matcher is capable of performing 1:1 and 1:n(few) matches with multipe biometric types. The following are the typical steps a matcher is expected to undertake.
* Perform a 1:1 match between sample and each of the gallery records. Each match operation results in a MatchDecision output. The result is an array of MatchDecision objects of the same size as the input parameter gallery.
* Each 1:1 match operation involves matchinf of multiple biometric types. Identify the list of biometric types for which the match is to be performed. This can be achieved by analysing the biometric segments in the sample input parameter and the modalitiesToMatch parameter.
* The match operation results in the creation of a MatchDecision object. This object contains a list of decisions. The decisions list has entries for each of the biometric types matched. For example if the modalities to match are iris and fingerprint, the decision list will have entries for those.
* The match process for each biometric type will result in a Decision object with the match result as "MATCHED", "NOT_MATCHED" or "ERROR". Additional analytics information in case of a decisive match result, and an error list in case of an error can be specified.
* A 1:n(few) match is seen essentially as multiple 1:1 match executed serially or parallely based on the SDK's threading capabilities.
* The matcher can return an error for unsupported biometric type and method in case it cannot match a particular biometric type.
* The actual match might work on Image-Image, Image-Extract or Extract-Extract combinations on what is present in the input segments in sample and gallery. When Extracts are used, they are either standard such as FMR or created by the SDK itself.

# Extractor

## Signature
`Response<BiometricRecord> extractTemplate(BiometricRecord sample, Map<String, String> flags)`

**Input Parameters**
* sample - Biometric Record of a person containing a list of biometric segments in the BIR format. The segments could correspond to multiple modalities and could be FIR, IIR, or Face Image Record.
* flags - An optional list of control flags as name value pairs that can be used to configure the behavior of the library.

**Output Parameters**
* BiometricRecord with extract Record in the form of FMR or a proprietary structure. The extract also contains a quality score. This contains extracts for all the segments passed in the input.

**Errors/Exceptions**
* Unsupported biometric type
* Unsupported image format
* Processing error

## Behavior
**Fingerprint**
* The extractor will extract either an ISO template FMR or extract a proprietary representation that can give better results. The extract record will be marked with the format and any additional metadata needed.

**Iris**
* Currently there are no known IRIS extraction standards. Any template extracted can be consumed only by a corresponding matcher.

**Face**
* Face analysis yields landmarks on face and these are typically stored in proprietary formats. Any template extracted can be used by a corresponding matcher.
{% hint style="info" %}
For non fingerprint biometrics characteristics and patterns and landmarks might be identified and stored in a custom format. This format will be proprietary to each extractor and can be only consumed by its corresponding matcher. The extract will also contain meta information about the extractor and the version of the algorithm it uses and any other assumption it has made in the process of extraction that can be useful during matching.
{% endhint %}

## Implementatin notes
The extractor is expected to create a data set that is used in the actual match process. This data set could be a standard format such as FMR in case fingerprint or a custom extract. The BDBInfo and BIRInfo variable should be filled in appropriately in the BiometricRecord returned. The signature block has to be filled with the hash computed and signed by the SDK. These extracts can be passed to the matcher.

# Segmenter

## Signature
`Response<BiometricRecord> segment(BIR sample, Map<String, String> flags)`

**Input Parameters**
* sample - Biometric Record of a person containing a list of BIR objects. The BIR would containt the unsegmented biometric in FIR, IIR, or Face Image Record.
* flags - An optional list of control flags as name value pairs that can be used to configure the behavior of the library.

**Output Parameters**
* BiometricRecord with list of segments created from the input unsegmented biometrics.

**Errors/Exceptions**
* Unsupported biometric type
* Unsupported image format
* Processing error

## Behavior per biometric type

**Fingerprint**
* Input will contain unsegmented image such as left slap or right slap or two thumbs
* Output will be biometrics of each finger present in the input image

**Iris**
* Input will contain unsegmented image such as both eyes
* Output will be biometrics of each eye present in the input image

**Face**
* Not applicable at present
{% hint style="info" %}
The segmenter will identify the individual fingers present.
{% endhint %}

## Implementation notes
The segmented biometrics should be stored in the BIR objects with the correct BDBInfo and BIRInfo attributes. Each segment should specify its biometric type, sub type. The signature block has to be filled with the hash computed and signed by the SDK.

# Converter

## Signature
`BiometricRecord convertFormat(BiometricRecord sample, String sourceFormat, String targetFormat, Map<String, String> sourceParams, Map<String, String> targetParams)`

**Input Parameters**
* sample - Biometric Record of a person containing a list of BIR objects. The BIR would containt the unsegmented biometric in FIR, IIR, or Face Image Record.
* sourceFormat - Specifies the input format. This could be JPEG, BMP, WSQ or other data formats based on the biometric type.
* sourceParams - An optional additional list that specifies more information about the source data such as "dpi", "fps", etc.
* targetFormat - Specifies the output format. This could be JPEG, BMP, WSQ or other data formats based on the biometric type.
* targetParams - An optional additional list that specifies more information about the output data such as "dpi", "fps", etc.

**Output Parameters**
* BiometricRecord with the biometric image data converted to the target format.

**Errors/Exceptions**
* Unsupported biometric type
* Unsupported image format
* Processing error

## Implementation notes
The converter will convert images in all segments in the biometric record. The signature block in the response has to be filled with the signed hash of the data.
{% hint style="info" %}
More documentation will be shared on the source and target params as well as source and target formats for standardaization. Review inputs are welcome.
{% endhint %}

# Operating Systems, Security and Other Considerations
* The SDK is specified as a Java SDK, but it could wrap C++/C implementations in a Java wrapper too.
* The SDKs are expected to run in Windows 10 as well as Linux (Ubuntu / CentOS) environments. Support for Android and IOS operating systems might be requested in future.
* The SDK must not rely on a specific deployment model as there are several ways they may be comsumed. Some are listed here below. This is not an exclusive list.
  * The application using the SDK can be deployed in a jvm instance running directly on the operating system like in the case of the registration client or in a virtual machine.
  * The service using the SDK can be deployed in a container such as a docker container in the case of authentication. Such docker instances might be created or destroyed on the fly.
  * Multiple instance of SDK might be running on the same physical hardware within VMs or containers.
* Some typical expectations from deployments with respect to security are as follows.
  * The SDK should typically not transfer data or consume data from any external sources. It should not access the network. It should be able to perform its duties even in the absence of a network.
  * The SDK should have its licensing checks that depend on parameters passed to it in the Init call and may not rely on or be tied to specific characteristic of the physical hardware, vm, or container instance it is running in.
  * The SDK is expected to be performant with sub second response times for 1:1 match and quality check and operate with a minimal footprint.
  * False Acceptance Rates and False Rejection Rates will need to meet benchmark expectations.
* Flags - Some mosip specific flags might be passed to method calls. These are to be logged. Specific flags supported by the SDK to tap specific behavior of the SDK can be documented and provided. The default behavior in the absence of flags should be specified and the optimal behavior should be the default behavior.
* Analytics - List of name value pairs that can be used to convey additional information. The values filled are specific to the implementing library. This could contain information about the aspects where quality is failing for e.g. ICAO compliance for tilt or lighting. In case of matches it could contain information like the NIST score, the algorithm used for matching and more.
* Errors - The error list can be used to list out specific causes of failure of the method called. This is useful for debugging and analysis.
* The Biometrics segments stored in BIR instances and the BIR class is modeled along the lines of CBEFF format and more information be found here: [Biometric Data Specification](Biometric-Data-Specification.md). 


# Appendix A - Java API Specifications
```java
// Enumerations
package io.mosip.kernel.biometrics.constant;

public enum BiometricFunction {
	QUALITY_CHECK,
	SEGMENT,
	EXTRACT,
	CONVERT_FORMAT,
	MATCH;
}

public enum BiometricType {	
	SCENT("Scent"),
  DNA("DNA"),
	EAR("Ear "),
	FACE("Face"),
	FINGER("Finger"),
	FOOT("Foot"),
	VEIN("Vein"),
	HAND_GEOMETRY("HandGeometry"),
	IRIS("Iris"),
	RETINA("Retina"), 
	VOICE("Voice"), 
	GAIT("Gait"),
	KEYSTROKE("Keystroke"), 
	LIP_MOVEMENT("LipMovement"), 
	SIGNATURE_SIGN("SignatureSign");
}

public enum Match {
	MATCHED,
	NOT_MATCHED,
	ERROR;	
}

public enum ProcessedLevelType {
	RAW("Raw"),
	INTERMEDIATE("Intermediate"),
	PROCESSED("Processed");
}

public enum PurposeType {	
	VERIFY("Verify"),
	IDENTIFY("Identify"),
	ENROLL("Enroll"),
	ENROLL_VERIFY("EnrollVerify"),
	ENROLL_IDENTIFY("EnrollIdentify"),
	AUDIT("Audit");
}

public class QualityType {
	RegistryIDType algorithm;
	Long score;
	String qualityCalculationFailed;
}

// Models and Entities

package io.mosip.kernel.biometrics.model;

public class Decision {	
	Match match;
	List<String> errors;
	Map<String, String> analyticsInfo;
}

public class MatchDecision {
	int galleryIndex;
	Map<BiometricType, Decision> decisions;
	Map<String, String> analyticsInfo;
}

public class QualityCheck {
	Map<BiometricType, QualityScore> scores;
	Map<String, String> analyticsInfo;
}

public class QualityScore {
	float score;	
	List<String> errors;
	Map<String, String> analyticsInfo;
}

class Response<T> {
	Integer statusCode;
	String statusMessage;
	T response;
}

public class SDKInfo {
  String apiVersion;
	String sdkVersion;
	List<BiometricType> supportedModalities;
	Map<BiometricFunction, List<BiometricType>> supportedMethods;
	Map<String, String> otherInfo;
	RegistryIDType productOwner;
}

// Entities continued
package io.mosip.kernel.biometrics.entities;

public class VersionType {
	int major;
	int minor;
}

public class SBInfo {
	private RegistryIDType format;
}

public class RegistryIDType {
	String organization;	
	String type;
}

public class BiometricRecord {
	VersionType version;
	VersionType cbeffversion;
	BIRInfo birInfo;
	List<BIR> segments;
}

public class BIRInfo {
	String creator;
	String index;
	byte[] payload;
	Boolean integrity;
	LocalDateTime creationDate;
	LocalDateTime notValidBefore;
	LocalDateTime notValidAfter;
}

public class BIR {
	VersionType version;
	VersionType cbeffversion;
	BIRInfo birInfo;
	BDBInfo bdbInfo;
	byte[] bdb;
	byte[] sb;
	SBInfo sbInfo;
	Map<String, Object> others;
}

public class BDBInfo {
	byte[] challengeResponse;
	String index;
	Boolean encryption;
	LocalDateTime creationDate;
	LocalDateTime notValidBefore;
	LocalDateTime notValidAfter;
	List<BiometricType> type;
	List<String> subtype;
	ProcessedLevelType level;
	RegistryIDType product;
	PurposeType purpose;
	QualityType quality;
	RegistryIDType format;
	RegistryIDType captureDevice;
	RegistryIDType featureExtractionAlgorithm;
	RegistryIDType comparisonAlgorithm;
	RegistryIDType compressionAlgorithm;
}

// Interface

package io.mosip.kernel.biometrics.spi;

public interface IBioApi {
{
  SDKInfo init(Map<String, String> initParams);
	Response<QualityCheck> checkQuality(BiometricRecord sample, List<BiometricType> modalitiesToCheck, Map<String, String> flags);
	Response<MatchDecision[]> match(BiometricRecord sample, BiometricRecord[] gallery, List<BiometricType> modalitiesToMatch, Map<String, String> flags);
	Response<BiometricRecord> extractTemplate(BiometricRecord sample, Map<String, String> flags);
	Response<BiometricRecord> segment(BIR sample, Map<String, String> flags);
	BiometricRecord convertFormat(BiometricRecord sample, String sourceFormat, String targetFormat, Map<String, String> sourceParams, Map<String, String> targetParams);
}
```

The above code snippets are available in - (https://github.com/mosip/commons/tree/1.0.9/kernel/kernel-biometrics-api/src/main/java/io/mosip/kernel/biometrics)

# Status Codes And Messages
Status Code	|Status Message	|Scenario
----- |----- |-----
2XX (range 200 to 299)	|OK |When everything is Okay
401	|Invalid Input Parameter - %s	|When data provided as input is invalid. (eg. Invalid Input Parameter - Gallery - FIR)
402	|Missing Input Parameter - %s	|When data required as input is missing. (eg. Missing Input Parameter - Probe - FIR)
403	|Quality check of Biometric data failed	|When data provided is valid but quality check cannot be performed
404	|Biometrics not found in CBEFF	|When there is no data found in the input CBEFF
405	|Matching of Biometric data failed	|When data provided is valid, but matching cannot be performed
406	|Data provided is of poor quality	|When some other error occurred (eg. licensing issue)
5XX (range 500 to 599)	|Unknown error occurred	|When some other error occurred (eg. licensing issue)
