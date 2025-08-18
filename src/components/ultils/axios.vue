<script>
    import axios from 'axios';

    const defaultUrl = 'http://localhost:8080';
    const defaultHeader = {};

    export function getFolders() {
        console.log('Get Folders Hierarchy');
        axios({
            method: 'get',
            url: defaultUrl + '/folder',
            headers: defaultHeader,
        })
        .then(response => {
            console.log('Get Folders Hierarchy success');
            return response.data;
        })
        .catch(error => {
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        });
    }

    export function postFolder() {
        console.log('Add new folder');
        axios({
            method: 'post',
            url: defaultUrl + '/folder',
            headers: defaultHeader,
            data: {
                id: null,
                name: 'New Folder',
            },
        })
        .then(response => {
            if (response.status === 201) {
                console.log('Add new folder success');
                return response.data;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            }
        })
        .catch(error => {
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        });
    }

    export function updateFolder(folderId, folderName) {
        console.log('Update Folder Name');
        axios({
            method: 'update',
            url: defaultUrl + '/folder/' + folderId,
            headers: defaultHeader,
            data: {
                id: folderId,
                name: folderName,
            }
        })
        .then(response => {if (response.status === 200) {
                console.log('Update Folder Name success');
                return true;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            } 
        })
        .catch(error => {
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        });
    }

    export function deleteFolder(folderId) {
        console.log('Delete Folder');
        axios({
            method: 'delete',
            url: defaultUrl + '/folder/' + folderId,
            headers: defaultHeader,
        })
        .then(response => {if (response.status === 200) {
                console.log('Delete Folder success');
                return true;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            } 
        })
        .catch(error => {
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        });
    }

    export function getFilesByFolderId(folderId) {
        console.log('Get Files by Folder ID');
        axios({
            method: 'get',
            url: defaultUrl + '/file/' + folderId,
            headers: defaultHeader,
        })
        .then(response => {
        console.log('Get Files by Folder ID success');
            return response.data;
        })
        .catch(error => {
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        });
    }

    export function postFile(fileName, folderId) {
        console.log('Post new file');
        axios({
            method: 'post',
            url: defaultUrl + '/file',
            headers: defaultHeader,
            data: {
                id: null,
                name: fileName,
                folderID: folderId,
            },
        })
        .then(response => {
            if (response.status === 201) {
                console.log('Post new file success');
                return response.data;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            }
        })
        .catch(error => {
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        });
    }

    export function updateFile(fileId, fileName, folderId) {
        console.log('Update File');
        axios({
            method: 'update',
            url: defaultUrl + '/file',
            headers: defaultHeader,
            data: {
                id: fileId,
                name: fileName,
                folderID: folderId,
            },
        })
        .then(response => {
            if (response.status === 200) {
                console.log('Update File success');
                return true;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            }
        })
        .catch(error => {
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        });
    }

    export function deleteFile(fileId, folderId) {
        console.log('Delete File');
        axios({
            method: 'delete',
            url: defaultUrl + '/file',
            headers: defaultHeader,
            data: {
                id: fileId,
                name: null,
                folderID: folderId,
            },
        })
        .then(response => {
            if (response.status === 200) {
                console.log('Delete File success');
                return true;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            }
        })
        .catch(error => {
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        });
    }

    export function getNote(fileId) {
        console.log('Get Note');
        axios({
            method: 'get',
            url: defaultUrl + '/note/' + fileId,
            responseType: 'blob',
            headers: defaultHeader,
        })
        .then(response => {
            console.log('Get Note success');
            return response.data.text();
        })
        .catch(error => {
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        });
    }

    export function postNote(fileId, noteContent) {
        console.log('Post new note');
        
        const chunkSize = 1000;
        const totalChunks = Math.ceil(noteContent.length / chunkSize);

        for (let i = 0; i < totalChunks; i++) {
            const chunk = noteContent.slice(i * chunkSize, (i + 1) * chunkSize);
            axios({
                method: 'post',
                url: defaultUrl + '/note/' + fileId,
                headers: defaultHeader,
                data: {
                    index: i,
                    total: totalChunks,
                    content: chunk,
                },
            })
            .then(response => {
                if (response.status === 201) {
                    console.log('Post new note chunk success');
                }
                else {
                    console.log(response.status, ' ', response.statusText);
                    return false;
                }
            })
            .catch(error => {
                if (error.response) {
                    console.error('HTTP error: ', error.response.data);
                } else {
                    console.error('Network error:', error.message);
                }
                return false;
            });
        }

        console.log('Post new note success');
        return true;
    }

    export function updateNote(fileId, noteContent) {
        console.log('Update Note');
        
        const chunkSize = 1000;
        const totalChunks = Math.ceil(noteContent.length / chunkSize);

        for (let i = 0; i < totalChunks; i++) { 
            const chunk = noteContent.slice(i * chunkSize, (i + 1) * chunkSize);
            axios({
                method: 'update',
                url: defaultUrl + '/note/' + fileId,
                headers: defaultHeader,
                data: {
                    index: i,
                    total: totalChunks,
                    content: chunk,
                },
            })
            .then(response => {
                if (response.status === 200) {
                    console.log('Update Note chunk success');
                }
                else {
                    console.log(response.status, ' ', response.statusText);
                    return false;
                }
            })
            .catch(error => {
                if (error.response) {
                    console.error('HTTP error: ', error.response.data);
                } else {
                    console.error('Network error:', error.message);
                }
                return false;
            });
        }

        console.log('Update Note success');
        return true;
    }

    export function deleteNote(fileId) {
        console.log('Delete Note');
        axios({
            method: 'delete',
            url: defaultUrl + '/note/' + fileId,
            headers: defaultHeader,
        })
        .then(response => {
            if (response.status === 200) {
                console.log('Delete Note success');
                return true;
            }
            else {
                console.log(response.status, ' ', response.statusText);
                return false;
            }
        })
        .catch(error => {
            if (error.response) {
                console.error('HTTP error: ', error.response.data);
            } else {
                console.error('Network error:', error.message);
            }
            return false;
        });
    }
</script>