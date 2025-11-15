#include <jni.h>
#include <opencv2/opencv.hpp>

extern "C"
JNIEXPORT jbyteArray JNICALL
Java_com_example_app_NativeBridge_processFrame(JNIEnv* env, jobject thiz, jbyteArray byteArr, jint width, jint height) {
    jbyte* data = env->GetByteArrayElements(byteArr, NULL);
    // Assuming RGBA input
    cv::Mat img(height, width, CV_8UC4, reinterpret_cast<unsigned char*>(data));

    cv::Mat gray, edges;
    cv::cvtColor(img, gray, cv::COLOR_RGBA2GRAY);
    cv::Canny(gray, edges, 50, 150);

    // Convert edges to RGBA for texture upload (white edges)
    cv::Mat outRGBA;
    cv::cvtColor(edges, outRGBA, cv::COLOR_GRAY2RGBA);
    int size = outRGBA.total() * outRGBA.elemSize();
    jbyteArray result = env->NewByteArray(size);
    env->SetByteArrayRegion(result, 0, size, reinterpret_cast<jbyte*>(outRGBA.data));
    env->ReleaseByteArrayElements(byteArr, data, JNI_ABORT);
    return result;
}
