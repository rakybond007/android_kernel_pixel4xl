# Pixel4XL_Kernel_build_script


## build script reference : github.com/GrapheneOS.kernel_google_coral
## 빌드과정 참고 구글 사이트 : https://source.android.com/setup/build/building?hl=ko

## AOSP 빌드 과정 (선행되어야함)
### 순서
환경 셋업 (envsetup, lunch) -> repo init  -> 아래 내용 1번 -> repo sync -> repo forall -c git lfs pull (opengapps 관련)-> vendor 설치 -> 빌드 (m)
### AOSP init
repo init -u https://android.googlesource.com/platform/manifest -b android-10.0.0_r40
repo init -u https://android.googlesource.com/platform/manifest -b android-11.0.0_r27 (픽셀5)
### repo sync 전 해야할 것.
1. opengapps 위해서 .repo/manifest.xml 수정하기 -> https://github.com/opengapps/aosp_build 참고
### repo sync 이후
2. vendor 설치 -> https://source.android.com/devices/automotive/start/pixelxl (픽셀4xl 경우 그대로 따라하기), https://developers.google.com/android/images 참고
-> https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds 여기서 버전태그 잘 확인하고 해야함. 드라이버 파일 아무거나 썼다가 픽셀5 계속 문제 남

픽셀4 xl

curl --output - https://dl.google.com/dl/android/aosp/google_devices-coral-qq3a.200705.002-8c59c992.tgz  | tar -xzvf -

tail -n +315 extract-google_devices-coral.sh | tar -zxvf -

curl --output - https://dl.google.com/dl/android/aosp/qcom-coral-qq3a.200705.002-316d4246.tgz | tar -xzvf -

tail -n +315 extract-qcom-coral.sh | tar -xzvf -

픽셀5 (RQ1A.210105.003)

curl --output - https://dl.google.com/dl/android/aosp/google_devices-redfin-rq1a.210105.003-53b67adf.tgz  | tar -xzvf -

tail -n +315 extract-google_devices-coral.sh | tar -zxvf -

curl --output - https://dl.google.com/dl/android/aosp/qcom-redfin-rq1a.210105.003-448ee17f.tgz | tar -xzvf -

tail -n +315 extract-qcom-coral.sh | tar -xzvf -

### 
After building AOSP, Set the AOSP path as ROOT_DIR

### Kernel Source && branch
git clone https://android.googlesource.com/kernel/msm
git checkout android-msm-coral-4.14-android10

## 실제 해보기
### 스크립트 실행 순서
make clean -> ./build_defconfig.sh -> ./build.sh
새롭게 빌드하려 할 때 out 디렉토리 지운 다음 하기.
### sys_call_table 주소 얻는법
1. grep "sys_call_table" kernel_dir/out/System.map
2. adb shell 접속, su 모드 -> echo 0 > /proc/sys/kernel/kptr_restrict -> cat /proc/kallsyms | grep sys_call_table

### 빌드한 커널로 새롭게 플래시하기
cp out/arch/arm64/boot/{dtbo.img,Image.lz4} "$ROOT_DIR/device/google/coral-kernel"
cp out/arch/arm64/boot/dts/google/qcom-base/sm8150.dtb "$ROOT_DIR/device/google/coral-kernel"
cp out/arch/arm64/boot/dts/google/qcom-base/sm8150-v2.dtb "$ROOT_DIR/device/google/coral-kernel"
위와 같이 3개의 파일 복사 -> envsetup, lunch 실행 (aosp 빌드할때 하는거, 돼있는 터미널이면 pass) -> m (빌드) -> adb reboot fastboot -> fastboot flashall
