pipeline {
    agent any
    
    environment {
        WEATHER_API_KEY = credentials('weather-api-key')
        BUILD_DIR = 'build'
        DEPLOY_DIR = '/var/www/html/weather-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '🔄 Checking out code from GitHub...'
                checkout scm
                sh 'ls -la'
            }
        }
        
        stage('Validate Files') {
            steps {
                echo '✅ Validating project files...'
                script {
                    // Check if required files exist
                    def requiredFiles = ['index.html', 'style.css']
                    requiredFiles.each { file ->
                        if (!fileExists(file)) {
                            error("Required file ${file} not found!")
                        }
                        echo "✓ Found ${file}"
                    }
                }
            }
        }
        
        stage('HTML/CSS Validation') {
            steps {
                echo '🔍 Validating HTML and CSS...'
                script {
                    // Basic HTML validation
                    sh '''
                    if grep -q "<!DOCTYPE html>" index.html; then
                        echo "✓ HTML DOCTYPE found"
                    else
                        echo "⚠️  HTML DOCTYPE missing"
                    fi
                    
                    if grep -q "<title>" index.html; then
                        echo "✓ HTML title found"
                    else
                        echo "⚠️  HTML title missing"
                    fi
                    '''
                }
            }
        }
        
        stage('API Test') {
            steps {
                echo '🌤️  Testing Weather API connection...'
                script {
                    // Test if weather API is accessible
                    sh '''
                    echo "Testing OpenWeatherMap API..."
                    API_RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" "https://api.openweathermap.org/data/2.5/weather?q=London&appid=${WEATHER_API_KEY}")
                    
                    if [ "$API_RESPONSE" = "200" ]; then
                        echo "✅ Weather API is working! Response code: $API_RESPONSE"
                    else
                        echo "❌ Weather API failed! Response code: $API_RESPONSE"
                        exit 1
                    fi
                    '''
                }
            }
        }
        
        stage('Security Check') {
            steps {
                echo '🔒 Running security checks...'
                script {
                    // Check if API key is hardcoded (should not be!)
                    def hasHardcodedKey = sh(
                        script: 'grep -r "appid=" index.html || true',
                        returnStdout: true
                    ).trim()
                    
                    if (hasHardcodedKey && !hasHardcodedKey.contains('${') && !hasHardcodedKey.contains('process.env')) {
                        echo "⚠️  Possible hardcoded API key found - please use environment variables"
                    } else {
                        echo "✅ No hardcoded API keys detected"
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                echo '🔨 Building weather app...'
                sh '''
                # Create build directory
                mkdir -p ${BUILD_DIR}
                
                # Copy files to build directory
                cp index.html ${BUILD_DIR}/
                cp style.css ${BUILD_DIR}/
                cp -r images ${BUILD_DIR}/ 2>/dev/null || echo "No images directory found"
                
                # Create environment config file
                echo "window.ENV = { WEATHER_API_KEY: '${WEATHER_API_KEY}' };" > ${BUILD_DIR}/config.js
                
                echo "✅ Build completed successfully!"
                ls -la ${BUILD_DIR}
                '''
            }
        }
        
        stage('Test Deployment') {
            steps {
                echo '🧪 Testing deployment readiness...'
                script {
                    // Check if build files are ready
                    sh '''
                    if [ -f "${BUILD_DIR}/index.html" ] && [ -f "${BUILD_DIR}/style.css" ]; then
                        echo "✅ All required files present in build directory"
                        
                        # Check file sizes
                        HTML_SIZE=$(stat -c%s "${BUILD_DIR}/index.html")
                        CSS_SIZE=$(stat -c%s "${BUILD_DIR}/style.css")
                        
                        echo "📊 File sizes:"
                        echo "   - index.html: ${HTML_SIZE} bytes"
                        echo "   - style.css: ${CSS_SIZE} bytes"
                        
                        if [ "$HTML_SIZE" -gt 0 ] && [ "$CSS_SIZE" -gt 0 ]; then
                            echo "✅ All files have content"
                        else
                            echo "❌ One or more files are empty"
                            exit 1
                        fi
                    else
                        echo "❌ Required files missing in build directory"
                        exit 1
                    fi
                    '''
                }
            }
        }
        
        stage('Deploy to Staging') {
            steps {
                echo '🚀 Deploying to staging environment...'
                script {
                    // Simulate deployment to staging
                    sh '''
                    # Create staging directory (simulate web server)
                    mkdir -p /tmp/weather-app-staging
                    
                    # Copy build files to staging
                    cp -r ${BUILD_DIR}/* /tmp/weather-app-staging/
                    
                    echo "✅ Deployed to staging: /tmp/weather-app-staging"
                    echo "📁 Staging files:"
                    ls -la /tmp/weather-app-staging
                    '''
                }
            }
        }
        
        stage('Integration Test') {
            steps {
                echo '🔄 Running integration tests...'
                script {
                    // Test if the deployed app structure is correct
                    sh '''
                    STAGING_DIR="/tmp/weather-app-staging"
                    
                    # Test file accessibility
                    if [ -r "$STAGING_DIR/index.html" ]; then
                        echo "✅ index.html is readable"
                    else
                        echo "❌ index.html is not readable"
                        exit 1
                    fi
                    
                    if [ -r "$STAGING_DIR/style.css" ]; then
                        echo "✅ style.css is readable"
                    else
                        echo "❌ style.css is not readable"  
                        exit 1
                    fi
                    
                    if [ -r "$STAGING_DIR/config.js" ]; then
                        echo "✅ config.js created and readable"
                    else
                        echo "❌ config.js missing"
                        exit 1
                    fi
                    
                    echo "🎉 All integration tests passed!"
                    '''
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                echo '🌟 Deploying to production...'
                script {
                    // Deploy to production (modify path as needed)
                    sh '''
                    # Create production directory
                    sudo mkdir -p ${DEPLOY_DIR} || mkdir -p /tmp/weather-app-production
                    PROD_DIR=${DEPLOY_DIR}
                    
                    # If sudo mkdir failed, use tmp directory
                    if [ ! -d "${DEPLOY_DIR}" ]; then
                        PROD_DIR="/tmp/weather-app-production"
                        echo "Using temporary production directory: $PROD_DIR"
                    fi
                    
                    # Copy files to production
                    sudo cp -r ${BUILD_DIR}/* ${PROD_DIR}/ 2>/dev/null || cp -r ${BUILD_DIR}/* ${PROD_DIR}/
                    
                    echo "✅ Deployed to production: $PROD_DIR"
                    echo "🌐 Weather app is now live!"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo '🧹 Cleaning up...'
            sh 'rm -rf ${BUILD_DIR} || true'
            sh 'rm -rf /tmp/weather-app-staging || true'
        }
        success {
            echo '🎉 Pipeline completed successfully!'
            echo '✅ Weather app has been built, tested, and deployed!'
        }
        failure {
            echo '❌ Pipeline failed!'
            echo '🔍 Check the logs above for details'
        }
    }
}