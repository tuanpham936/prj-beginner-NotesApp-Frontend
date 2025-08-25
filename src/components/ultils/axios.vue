<script>
    import axios from 'axios';

    const defaultUrl = 'http://localhost:8080';
    const defaultHeader = {};

    export async function getFolders() {
        try {
            console.log('Get Folders Hierarchy');
            const response = await axios({
                method: 'get',
                url: defaultUrl + '/folder',
                headers: defaultHeader,
            })
            console.log('Get Folders Hierarchy success');
            return response.data;
        }
        catch(error) {
            console.log('Get Folders Hierarchy failed');
            if (error.response) {
                console.error(error.response.data);
            } else {
                console.error(error.message);
            }
            return false;
        };
    }

    export async function postFolder() {
        try {
            console.log('Add new folder');
            const response = await axios({
                method: 'post',
                url: defaultUrl + '/folder',
                headers: defaultHeader,
                data: {
                    id: null,
                    name: 'New Folder',
                },
            })
            
            if (response.status === 201) {
                console.log('Add new folder success');
                return response.data;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            }
        }
        catch(error) {
            console.log('Add new folder failed');
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        };
    }

    export async function updateFolder(folderId, folderName) {
        try {
            console.log('Update Folder Name');
            const response = await axios({
                method: 'put',
                url: defaultUrl + '/folder/' + folderId,
                headers: defaultHeader,
                data: {
                    id: folderId,
                    name: folderName,
                }
            })
            if (response.status === 200) {
                console.log('Update Folder Name success');
                return true;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            } 
        }
        catch(error) {
            console.log('Update Folder Name failed');
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        };
    }

    export async function deleteFolder(folderId) {
        try {
            console.log('Delete Folder');
            const response = await axios({
                method: 'delete',
                url: defaultUrl + '/folder/' + folderId,
                headers: defaultHeader,
            })
            if (response.status === 200) {
                console.log('Delete Folder success');
                return true;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            } 
        }
        catch(error) {
            console.log('Delete Folder failed');
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        };
    }

    export async function getFilesByFolderId(folderId) {
        try {
            console.log('Get Files by Folder ID');
            const response = await axios({
                method: 'get',
                url: defaultUrl + '/file/' + folderId,
                headers: defaultHeader,
            })
            console.log('Get Files by Folder ID success');
            return response.data;
        }
        catch(error) {
            console.log('Get Files by Folder ID failed');
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        };
    }

    export async function postFile(fileName, folderid) {
        try {
            console.log('Post new file');
            const response = await axios({
                method: 'post',
                url: defaultUrl + '/file',
                headers: defaultHeader,
                data: {
                    id: null,
                    name: fileName,
                    folderId: folderid,
                },
            })
            if (response.status === 201) {
                console.log('Post new file success');
                return response.data;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            }
        }
        catch(error) {
            console.log('Post new file failed');
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        };
    }

    export async function updateFile(fileId, fileName, folderid) {
        try {
            console.log('Update File');
            const response = await axios({
                method: 'put',
                url: defaultUrl + '/file',
                headers: defaultHeader,
                data: {
                    id: fileId,
                    name: fileName,
                    folderId: folderid,
                },
            })
            if (response.status === 200) {
                console.log('Update File success');
                return true;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            }
        }
        catch(error) {
            console.log('Update File failed');
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        };
    }

    export async function deleteFile(fileId, folderid) {
        try {
            console.log('Delete File');
            const response = await axios({
                method: 'delete',
                url: defaultUrl + '/file',
                headers: defaultHeader,
                data: {
                    id: fileId,
                    name: null,
                    folderId: folderid,
                },
            })
            if (response.status === 200) {
                console.log('Delete File success');
                return true;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            }
        }
        catch(error) {
            console.log('Delete File failed');
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        };
    }

    export async function getNote(fileId) {
        try {
            console.log('Get Note');
            const response = await axios({
                method: 'get',
                url: defaultUrl + '/note/' + fileId,
                responseType: 'blob',
                headers: defaultHeader,
            })
            console.log('Get Note success');
            return response.data.text();
        }
        catch(error) {
            console.log('Get Note failed');
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        };
    }

    export async function postNote(fileId, noteContent) {
        try {
            console.log('Post new note');
            
            const blob = new Blob([noteContent], { type: "text/plain" });
            const file = new File([blob], fileId + ".txt", { type: "text/plain" });

            const formData = new FormData();
            formData.append("note", file);

            const response = await axios({
                method: 'post',
                url: defaultUrl + '/note/' + fileId,
                headers: defaultHeader,
                data: formData,
            })
            if (response.status === 201) {
                console.log('Post new note success');
                return true;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            }
        }
        catch(error) {
                console.log('Post new note failed');
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        }        
    }

    export async function updateNote(fileId, noteContent) {
        try {
            console.log('Update Note');
            
            const blob = new Blob([noteContent], { type: "text/plain" });
            const file = new File([blob], fileId + ".txt", { type: "text/plain" });

            const formData = new FormData();
            formData.append("note", file);
        
            const response = await axios({
                method: 'put',
                url: defaultUrl + '/note/' + fileId,
                headers: defaultHeader,
                data: formData,
            });

            if (response.status === 200) {
                console.log('Update Note success');
                return true;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            }
        }
        catch(error) {
            console.log('Update Note failed');
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        };
    }
    
    // export function deleteNote(fileId) {
    //     console.log('Delete Note');
    //     axios({
    //         method: 'delete',
    //         url: defaultUrl + '/note/' + fileId,
    //         headers: defaultHeader,
    //     })
    //     .then(response => {
    //         if (response.status === 200) {
    //             console.log('Delete Note success');
    //             return true;
    //         }
    //         else {
    //             console.log(response.status, ' ', response.statusText);
    //             return false;
    //         }
    //     })
    //     .catch(error => {
    //         if (error.response) {
    //             console.error('HTTP error: ', error.response.data);
    //         } else {
    //             console.error('Network error:', error.message);
    //         }
    //         return false;
    //     });
    // }
</script>