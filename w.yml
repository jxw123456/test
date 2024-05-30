stages:          
  - build
  - post
  - sendlog
  
build-job:       
  stage: build
  before_script:
    - chcp 65001
    - pip install openpyxl

  script:
    - git checkout $CI_COMMIT_BRANCH
    - git fetch origin
    - git reset --hard origin/$CI_COMMIT_BRANCH
    - git clean -fd
    
    - echo Building.......
    - python prebuild.py
    - cd .\Application\Build\Tools
    - .\Clean
    - .\Build
    - cd ..
    - if(-not (Test-path .\Bin\BMS.hex)) {echo "build failed"; exit 1}
    - cd ..\Tools\StackCounter\
    - python StackCounter.py
    - Move-Item -Path "Stack_Length.xlsx" -Destination "..\..\Build\Bin\Stack_Length.xlsx"
    - cd ..\..\Build
    - .\PostBuildGenSignatureFlashHex
    - .\Hex2S19
    - cd ..\A2LGen
    - echo Generating A2L......
    - python main_ape.py
    - python main_inca.py
    - Move-Item -Path "BMS_App_Ape.a2l" -Destination "..\Build\Bin\BMS_App_Ape.a2l"
    - Move-Item -Path "BMS_App_Inca.a2l" -Destination "..\Build\Bin\BMS_App_Inca.a2l"
    
    - cd $CI_PROJECT_DIR
    - git add .
    - git stash

  rules:
    - if: $CI_COMMIT_MESSAGE =~ /\[ci\]|\[ci\+\]/
      when: always
    - when: manual

  tags:
    - bms_build_runner_cloud

post-build:   
  stage: post    # It only starts when the job in the build stage completes successfully.
  script:
    - echo CI_COMMIT_BRANCH=${CI_COMMIT_BRANCH}
    - echo CI_COMMIT_TITLE=$CI_COMMIT_TITLE
    - echo CI_PROJECT_DIR=$CI_PROJECT_DIR    
    - $CI_COMMIT_AUTHORNAME=$(git log --format="%an" -n 1 "${CI_COMMIT_SHA}")
    - $CI_COMMITER_EMAIL=$(git log -1 $CI_COMMIT_SHA --pretty="%cE")
    - $BUILD_STATUS="Pass"
    - echo $CI_COMMIT_AUTHORNAME
    - echo $CI_COMMITER_EMAIL
    - git checkout $CI_COMMIT_BRANCH
    - git fetch origin
    - git reset --hard origin/$CI_COMMIT_BRANCH
    - git clean -fd
    - git stash pop

    - cd .\Application\Build\Bin
    - if(Test-path BMS_App.7z) {Remove-Item BMS_App.7z}
    - py7zr c BMS_App.7z BMS.elf BMS.map BMS.hex BMS_Flash.hex BMS_App_Ape.a2l BMS_App_Inca.a2l BMS_Flash.s19 Stack_Length.xlsx
    - cd $CI_PROJECT_DIR
    #- Remove-Item BMS.elf BMS.map BMS.hex BMS_Flash.hex BMS_App.a2l

    - if ($CI_COMMIT_TITLE -match "\[ci\]"){
        echo "sending packcage only";        
        dir;
        python postbuild.py $CI_COMMITER_EMAIL $CI_COMMIT_TITLE $BUILD_STATUS
        }
      else{
        git config --global user.name "Gitlab Runner";
        git config --global core.quotepath false;
        git config --global --add safe.directory $CI_PROJECT_DIR;
        git status;
        git add .;
        git commit -m "CICD:$CI_COMMIT_TITLE [skip ci]";
        git push -o ci-skip https://gitlab-ci-token:$RunnerToken@gitlab.chehejia.com/$GitPath;
        $COMMITID=$(git log -1 --format="%H");        
        python postbuild.py $CI_COMMITER_EMAIL $CI_COMMIT_TITLE $BUILD_STATUS $COMMITID
        }

  rules:
    - when: on_success      
      
  dependencies:
    - build-job
  tags:
    - bms_build_runner_cloud
  
failure-reaction:
  stage: sendlog
  before_script:
    - chcp 65001

  script:
    - $CI_COMMIT_AUTHORNAME=$(git log --format="%an" -n 1 "${CI_COMMIT_SHA}")
    - $CI_COMMITER_EMAIL=$(git log -1 $CI_COMMIT_SHA --pretty="%cE")
    - $BUILD_STATUS="Fail"
    - echo $CI_COMMIT_AUTHORNAME
    - echo $CI_COMMITER_EMAIL    
    - cd $CI_PROJECT_DIR
    - python postbuild.py $CI_COMMITER_EMAIL $CI_COMMIT_TITLE $BUILD_STATUS
  rules:
      - when: on_failure

  dependencies:
    - build-job
  tags:
    - bms_build_runner_cloud
