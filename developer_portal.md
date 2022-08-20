# Pi Developer Portal

With the help of Pi SDK and Pi API, you can create applications that run within the Pi browser, which can be used by other Pioneers. Pi Developer Portal is where you can register and configure your apps so that it can be connected to the Pi Blockchain.

## How to register your app

1. The Pi Developer Portal can be accessed using the Pi Browser. Go to `pi://develop.pi` to visit the Pi Developer Portal. To register a new app, click on `New App` button at the bottom of the page.

<div style="text-align:center;">
    <img src="./img/dev_portal_main_1.png" alt="drawing" width="200"/>
</div>

2. You can fill in the required fields. One important field is selecting the host network that the app is connecting to. Depending on this option, Pi SDK will automatically connect your app to the corresponding network. An app can only connect to one network at a time, and once you register the app, this option cannot be changed.

<div style="text-align:center;">
    <img src="./img/dev_portal_new_1.png" alt="drawing" width="200"/>
</div>

> When the app is connecting to the Pi Testnet, you can maintain a list of whitelisted users. Your testnet app will only be accessible by users you allowed, and if you leave this field empty, anyone can access your test app.

<div style="text-align:center;">
    <img src="./img/dev_portal_new_2.png" alt="drawing" width="200"/>
</div>

> When the app is connecting to the Pi Mainnet, you can pair one of your testnet apps to the mainnet app. More specific usage is coming soon in the future.

> When you're hosting the app on your server, you will also need to provide the url. Notice that you'll be asked to verify the ownership of the url after you register the app. After you filled in the required fields, you can submit to register your app by clicking on `Submit` button at the bottom of the page.

<div style="text-align:center;">
    <img src="./img/dev_portal_new_3.png" alt="drawing" width="200"/>
</div>

3. After you register your app, you can see your apps in the main page. If you selected the paired app for your mainnet app, it will show like the following.

<div style="text-align:center;">
    <img src="./img/dev_portal_main_2.png" alt="drawing" width="200"/>
</div>
<br>

## Testnet app in the Pi Browser

If you registered your app to connect to the Pi Testnet and if you visit your app in the Pi Browser, it will show the black and yellow stripe at the top to indicate that this app is connected to the Pi Testnet.

<div style="text-align:center;">
    <img src="./img/testnet_indicator.png" alt="drawing" width="200"/>
</div>


## Legacy Project created before 26/07/2022 

Projects that existed prior to the launch of the Pi Mainnet SDK will point to the Pi Testnet
- This cannot be changed or updated
- It is best practice to create new Developer Portal Projects

### Developers and Teams with previously created Legacy Projects

1. Create a new Developer Portal Project 
    - Select Mainnet - This will be the Mainnet Version of the App
    - URL
        - This project should contain the desired URL that Pioneers will access the app with through the Pi Browser
        - URL ownership will be determined through verification process
    - Payment Wallet
        - Until developer/app wallets are created the wallet address of the developer who creates the new project page will be used to process all transactions
        - If the project page creator does not have a Pi Wallet Address which has been created in a Mainnet Migration all transactions will fail.  
2. Create a second new Developer Project Page
    - Select Testnet - This will be for testing purposes
    - URL
        - Cannot be the same as the Mainnet URL
        - Must be owned by the developer as well
    - Set the desired access permissions 
    - This project can be linked to the previously created Mainnet Project from step 1
3. Link Mainnet Project to the Brainstorm project (if applicable)
    - As of July 2022 it is possible to link only one developer portal project to a brainstorm project

### Special Considerations
- If an app team has no members with a Mainnet Wallet
    - Please reach out to the Pi CT through the <a href="https://pinetwork.atlassian.net/servicedesk/customer/portal/1/group/3/create/20" target="_blank">Pi Support Portal</a>
    - For the question “What does your question relate to? ”
        - Select “Pi Mainnet SDK Wallet”
        - Submit contact information
        - A member of the Pi CT will follow up
