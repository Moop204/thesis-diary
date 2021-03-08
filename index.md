# Welcome to Moop204's Thesis Diary

## Whats the Thesis? 
[CellML](https://www.cellml.org/) is a markup language that aims to describe biological structures in a set standard. My goal is to create a CellML Editor that is more user friendly and intuitive to use in order to attract more people to adopt this standard. The only competitor in a pure CellML editor is [OpenCOR](https://opencor.ws/) which has a large set of features and is developed by a group of people. In order to compete with the established editor this project will adopt the same library to parse and modify CellML files ([libcellml](https://github.com/cellml/libcellml)). 

## Technology Stack 
This will be made in [Electron](https://www.electronjs.org/) using [React](https://reactjs.org/) for the frontend and [Typescript](https://www.typescriptlang.org/) all round. This project will make heavy use of Electron's [IPC](https://www.electronjs.org/docs/api/ipc-main) in order to communicate to libcellml from the frontend. To make everything look consistent the React-friendly UI framework [Material-UI](https://material-ui.com/) is used.  

---

## Building Libcellml
I learned a very important lesson during my attempts to build the libcellml library. Its a lot less painful to ask for help than to try everything, fail and then ask for help. 

Looking at the libcellml source there was a cmake which I had seen plenty of times and built with a few times. This was when I severely underestimated cmake. I was stuck for days trying to understand make files, cmake and the error messages that kept popping up. Eventually I realised that it kept complaining about libxml2 and SWIG. This was confusing to me because my system had both of those libraries already installed. 

The documentation for libcellml was bare as it was still under development. The build instructions covered what I did but mostly focused on a Windows system. Trying it gave no result. In the meantime I ended up being in contact with the libcellml developers and asked them for help regarding how to use libcellml. They redirected me to [hsorby](https://github.com/hsorby) who showed me the emscripten fork of libcellml he was developing. After more days of trying I gave up and asked hsorby how they had managed to build his bindings with libcellml. In just a few days he had written great documentation detailing the installation of libcellml including the two dependencies I was missing as well as how to build the webassembly file needed for my project. 

## Configuring Project
WebAssembly is a pain. Webpack is a pain. Both of them together is very painful. 

Based on advice from a friend I used [Electron Forge](https://www.electronforge.io/) which configured an Electron application all for you in a few lines. I used a React based setup which worked out very well. It ran great, additional packages could be installed without a problem and I was able to develop a bit of the user interface. As I began to implement more functionality I had to start using the libcellml. Thats where everything went wrong. 

Libcellml existed as a WebAssembly file with an accompanying .js file. I knew nothing of how to use either of them. Most results gave answers for Rust-based projects in WebAssembly rather than C++. One of the dumber problems was the asynchronous problem. Most of the sample code online used fetch or fs which looked for local files from browser JS or NodeJS respectively. Its embarassing to say but I did not know how to use async functions properly. It took hundreds of searches until I realised that an await at the top level was not allowed by Babel. I assumed it was some configuration issue. Turns out it was actually because I was using await at the top level. The standard practice in that case is to throw everything inside a giant async function and call it which worked out great. 

The problem eventually came to be the inability for Webpack to acknowledge the existence of the .wasm file. I tried to use WebAssembly loaders, even the outdated 3+ years since it was updated ones. I was adamant that the problem was in the WebPack configurations. Eventually I came past a Stack Overflow post that talked about how WebPack v5 had better support for WebAssembly than WebPack v4. This was a problem. ElectronForge which configured the entire project used its own version of WebPack which was only on v4. This led me scrambling for another configuration for my project. My saviour came in the form of [Electron React Boilerplate](https://github.com/electron-react-boilerplate/electron-react-boilerplate), despite its unwieldly name and messier output it had WebPack v5. After a bit of copying and pasting it successfully imported and used the WebAssembly file!

