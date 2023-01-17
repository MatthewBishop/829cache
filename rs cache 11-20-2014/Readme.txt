This is an archive with unpacked and decompressed reference table containers. They were dumped in the following code from RSCD:

	private void unpack(int index, byte[] data, ReferenceTable main) {
	//	try {
	//		CacheDownloader.writeFile(index+".crc", ByteBuffer.wrap(data));
	//	} catch (IOException e) {
	//		// TODO Auto-generated catch block
	//		e.printStackTrace();
	//	}
		ByteBuffer buffer = ByteBuffer.wrap(data);
		
		protocol = buffer.get() & 0xff;
		if (protocol >= 6) {
            version = buffer.getInt();
            if (main != null) {
                Entry entry = main.getEntries()[index];
                if (entry.version != version) {
                    throw new RuntimeException("Version mismatch " + index + "," + version + "," + entry.version);
                }
            }
        }
        int flags = buffer.get() & 0xff;
        hasIdentifiers = (flags & 0x1) != 0;
		hasDigests = (flags & 0x2) != 0;
        boolean flag4 = (flags & 0x4) != 0;
        boolean flag8 = (flags & 0x8) != 0;

		if (protocol >= 7) {
			entryCount = getSmart(buffer);
		} else {
			entryCount = buffer.getShort() & 0xffff;
		}
		entries = new Entry[entryCount];

		int off = 0;
		int count = -1;
		for (int i = 0; i < entryCount; i++) {
			Entry entry = entries[i] = new Entry();
			entry.index = off += protocol >= 7 ? getSmart(buffer) : (buffer.getShort() & 0xffff);
			if (entry.index > count) {
				count = entry.index;
			}
		}

		if (hasIdentifiers) {
			for (int i = 0; i < entryCount; i++) {
				entries[i].identifier = buffer.getInt();
			}
		} else {
            for (int i = 0; i < entryCount; i++) {
				entries[i].identifier = -1;
			}
        }

		for (int i = 0; i < entryCount; i++) {
			entries[i].crc = buffer.getInt();
		}

        if (flag8) {
            for (int i = 0; i < entryCount; i++) {
                int v = buffer.getInt();
            }
        }

		if (hasDigests) {
			for (int i = 0; i < entryCount; i++) {
				Entry entry = entries[i];
				entry.digest = new byte[64];
				buffer.get(entry.digest);
			}
		}

        if (flag4) {
            for (int i = 0; i < entryCount; i++) {
                int v1 = buffer.getInt();
                int v2 = buffer.getInt();
            }
        }

		for (int i = 0; i < entryCount; i++) {
			entries[i].version = buffer.getInt();
		}

		for (int i = 0; i < entryCount; i++) {
			entries[i].childCount = protocol >= 7 ? getSmart(buffer) : (buffer.getShort() & 0xffff);
		}

		for (int i = 0; i < entryCount; i++) {
			Entry entry = entries[i];
			off = 0;
			int children = entry.childCount;
			entry.childIndices = new int[children];
			count = -1;
			for (int j = 0; j < children; j++) {
				entry.childIndices[j] = off += protocol >= 7 ? getSmart(buffer) : (buffer.getShort() & 0xffff);
				if (entry.childIndices[j] > count) {
					count = entry.childIndices[j];
				}
			}
			entry.childIndexCount = count + 1;
			if (count + 1 == children) {
				entry.childIndices = null;
			}
		}

		if (hasIdentifiers) {
			for (int i = 0; i < entryCount; i++) {
				Entry entry = entries[i];
				int children = entry.childCount;
				entry.childIdentifiers = new int[entry.childIndexCount];
				for (int j = 0; j < entry.childIndexCount; j++) {
					entry.childIdentifiers[j] = -1;
				}
				for (int j = 0; j < children; j++) {
					int k;
					if (entry.childIndices != null) {
						k = entry.childIndices[j];
					} else {
						k = j;
					}
					entry.childIdentifiers[k] = buffer.getInt();
				}
			}
		}
	}

